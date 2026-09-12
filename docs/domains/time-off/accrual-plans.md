# Time Off — Accrual Plans

An Accrual Plan turns a Time Off Allocation into a growing entitlement. Instead of granting a
fixed number of days once, the allocation is advanced by a scheduled process that walks period
boundary after period boundary, adding the amount configured for the milestone in force, clipping
the result against a running cap and a yearly cap, applying the carry-over policy at a yearly
cut-off, expiring carried-over entitlement after a validity window, and switching from one
milestone to the next as the employee gains seniority.

This file specifies the whole machinery: the configuration surface, the selection of the level in
force, the period boundary functions of every frequency, the proration rules, the engine as one
ordered procedure, the projection that answers "how much will this allocation have by then", and
five long worked examples with real dates and real numbers.

The entity field tables are in [entities.md, chapter 6](entities.md#6-accrual-plan) and
[entities.md, chapter 7](entities.md#7-accrual-plan-level); the validations are rules `TOF-080` to
`TOF-089` of [business-rules.md](business-rules.md#8-accrual-plans-and-levels); the cursor fields
are in [state-machines.md, chapter 4](state-machines.md#4-the-accrual-cursor).

---

## 1. Vocabulary

| Term | Meaning |
|---|---|
| Accrual Plan | The named configuration attached to an allocation. |
| Accrual Plan Level, called a *milestone* on the screens | One stage of the plan. It starts a fixed offset after the allocation's validity start date and grants a fixed amount at a fixed frequency. |
| Accrual period | The interval between two consecutive boundaries of the level in force, for example one calendar month for a monthly level. |
| Accrual date | A date on which a period closes and a grant lands. |
| Carry-over cut-off, also called the carry-over anchor | The yearly date on which the carry-over policy is applied. |
| Level transition date | The date on which a level starts: the allocation's validity start date plus the level's offset. |
| Last call (`lastcall`) | The date of the last period boundary on which a grant landed. |
| Actual last call (`actual_lastcall`) | The date of the last engine iteration, whether or not it granted anything. |
| Next call (`nextcall`) | The next date on which the engine must act on this allocation. |
| Carried-over pool (`expiring_carryover_days`) | The balance recorded on a carry-over cut-off, which is the amount that may later expire. |
| Gross balance (`number_of_days`) | The running accrued amount of the allocation, including the part already consumed by absence. |
| Available balance | The gross balance minus the consumed amount. This is what a cap caps. |

---

## 2. The configuration surface

### 2.1 Plan settings

| Setting | Values | Effect |
|---|---|---|
| Accrued Gain Time (`accrued_gain_time`) | `start` "At the start of the accrual period", `end` "At the end of the accrual period" | Whether a period's grant lands when the period opens or when it closes. Granting at the start makes the engine pre-grant the period that is currently open and marks the allocation so that the same period is not granted twice. |
| Based on worked time (`is_based_on_worked_time`) | yes, no | When yes, the grant is multiplied by the share of the period the employee actually worked. Forced to no whenever the grant lands at the start of the period. |
| Carry-Over Time (`carryover_date`) | `year_start` "At the start of the year", `allocation` "At the allocation date", `other` "Custom date" | Which yearly anchor triggers the carry-over evaluation. |
| Carry-over month and day (`carryover_month`, `carryover_day`) | a month, and a day clipped to that month's length in a leap year | Used by the custom anchor only. |
| Milestone Transition (`transition_mode`) | `immediately` "Immediately", `end_of_accrual` "After this accrual's period" | Whether a level change takes effect on the level transition date or only once the running period has closed. Shown on the form only when the plan carries more than one milestone. |
| Time Off Type (`time_off_type_id`) | a Time Off Type, or empty | When set, the plan may only be used with that type, and the grant unit of every level is forced to days for a day-based or half-day-based type and to hours for an hour-based type. |
| Value unit (`added_value_type`) | `day` "Days", `hour` "Hours" | Mirrors the first level's grant unit onto the plan and onto every other level. |
| Can be carried over (`can_be_carryover`) | yes, no | When no, every level is forced to the "Lost" carry-over action. |

### 2.2 Milestone settings

| Setting | Values | Effect |
|---|---|---|
| Milestone reached (`milestone_date`, `start_count`, `start_type`) | "At allocation creation", or "After" a number of Days, Months or Years | The level transition date, counted from the allocation's validity start date. An offset of zero means "At allocation creation". |
| Rate (`added_value`, `added_value_type`) | a strictly positive decimal with five decimal places, and a unit of Day(s) or Hour(s) | How much lands each period. |
| Frequency (`frequency`) | `hourly` "Hourly", `daily` "Daily", `weekly` "Weekly", `bimonthly` "Twice a month", `monthly` "Monthly", `biyearly` "Twice a year", `yearly` "Yearly", and, with the attendance companion package, `worked_hours` "Per Hour Worked" | How often a period closes. |
| Allocation on (`week_day`) | Monday through Sunday | The anchor of a weekly frequency. |
| First day, second day (`first_day`, `second_day`) | 1 to 31 | The anchors of a twice-a-month frequency; the first day alone is the anchor of a monthly frequency. |
| First month and day, second month and day (`first_month`, `first_month_day`, `second_month`, `second_month_day`) | January to June with a day, July to December with a day | The anchors of a twice-a-year frequency. |
| Yearly month and day (`yearly_month`, `yearly_day`) | any month with a day | The anchor of a yearly frequency. |
| Cap accrued time (`cap_accrued_time`, `maximum_leave`) | off, or a strictly positive amount | The available balance never rises above this amount. |
| Cap accrued time yearly (`cap_accrued_time_yearly`, `maximum_leave_yearly`) | off, or a strictly positive amount | The total granted between two carry-over cut-offs never rises above this amount. |
| Unused accruals (`action_with_unused_accruals`) | `lost` "Lost", `all` "Carried over" | What happens to the unused balance at the carry-over cut-off. |
| Carry-over options (`carryover_options`, `postpone_max_days`) | `unlimited` "Unlimited", or `limited` "Up to" a strictly positive number | How much of the unused balance survives the cut-off. |
| Carried-over validity (`accrual_validity`, `accrual_validity_count`, `accrual_validity_type`) | off, or a count and a unit of Days or Months | How long the carried-over pool stays usable after the cut-off. |

The ordering of the milestones is derived, never typed:

```formula
sequence = start count × multiplier of the start unit
```

with a multiplier of one for days, thirty for months and three hundred and sixty-five for years.
Two milestones with the same derived key are ordered arbitrarily; a plan should not carry two
milestones with the same offset.

---

## 3. Selecting the level in force on a date

**Input**: an allocation, which carries its validity start date and its plan, and a date.
**Output**: a level and its index in the ordered list, or (none, none) when the plan carries no
level.

1. A plan with no level yields (none, none).
2. Order the levels by the derived sequence ascending.
3. Walk the list from first to last, keeping the **last** level whose level transition date is
   **strictly before** the given date, starting from (none, index minus one). The level transition
   date is the allocation's validity start date plus the level's offset in the level's start unit.
4. When the retained index is zero or below, **or** the plan transitions immediately, the retained
   pair is the answer.
5. Otherwise the plan transitions at the end of the accrual period. Let the *level start* be the
   retained level's transition date and the *previous level* be the one just before it in the list.
   When the retained level's next boundary computed from the level start falls **strictly before**
   the previous level's next boundary computed from the same date, the answer is the previous level
   and its index; otherwise it is the retained pair.

Step 5 is what implements the "After this accrual's period" transition: the previous level keeps
the allocation until the period it had opened would have closed.

The comparison in step 3 is strict. On the exact level transition date the level is not yet in
force; it becomes in force the day after. The initialisation of the cursor compensates for this by
treating a first level whose transition date is today as the level in force, which is specified in
[entities.md, section 5.12](entities.md#512-initialisation-of-the-accrual-cursor-at-creation).

---

## 4. Period boundaries

Two pure functions of a date define the accrual periods: the **next boundary** strictly after a
date, and the **previous boundary** at or before a date. Both are total and deterministic; an
unrecognised frequency raises, from either of them, *"Your frequency selection is not correct:
please choose a frequency between theses options:Hourly, Daily, Weekly, Twice a month, Monthly,
Twice a year and Yearly."*

Four helpers are used:

| Helper | Meaning |
|---|---|
| with day( date , number ) | the date with the same year and month and the day set to the smaller of that number and the length of that month |
| with month and day( date , month , number ) | the date with the same year, that month, and the day set to the smaller of that number and the length of that month in that year |
| add months( date , count ), add years( date , count ) | calendar arithmetic that clamps the day to the length of the target month |
| forward to weekday( date , weekday ) | the date itself when it already falls on that weekday, otherwise the next date after it that does |

| Frequency | Next boundary strictly after the date | Previous boundary at or before the date |
|---|---|---|
| Hourly, Daily, Per Hour Worked | the date plus one day | the date itself |
| Weekly | forward to weekday( the date plus one day , the configured weekday ) | forward to weekday( the date minus six days , the configured weekday ) |
| Twice a month | let *first* be with day( the date , the first day ) and *second* be with day( the date , the second day ); *first* when the date precedes *first*; *second* when the date precedes *second*; otherwise with day( add months( the date , 1 ) , the first day ) | *second* when the date is at or after *second*; *first* when the date is at or after *first*; otherwise with day( add months( the date , −1 ) , the second day ) |
| Monthly | let *candidate* be with day( the date , the first day ); *candidate* when the date precedes it; otherwise with day( add months( the date , 1 ) , the first day ) | *candidate* when the date is at or after it; otherwise with day( add months( the date , −1 ) , the first day ) **plus one day** |
| Twice a year | let *first* be with month and day( the date , the first month , the first month day ) and *second* likewise with the second pair; *first* when the date precedes *first*; *second* when the date precedes *second*; otherwise with month and day( add years( the date , 1 ) , the first month , the first month day ) | *second* when the date is at or after *second*; *first* when the date is at or after *first*; otherwise with month and day( add years( the date , −1 ) , the second month , the second month day ) |
| Yearly | let *candidate* be with month and day( the date , the yearly month , the yearly day ); *candidate* when the date precedes it; otherwise with month and day( add years( the date , 1 ) , the yearly month , the yearly day ) | *candidate* when the date is at or after it; otherwise with month and day( add years( the date , −1 ) , the yearly month , the yearly day ) |

Two properties matter to the engine and must be reproduced exactly.

- The next boundary is **strictly forward**: given a date that is itself a boundary it yields the
  following boundary. The previous boundary is **inclusive**: given a date that is itself a
  boundary it yields that same date.
- The monthly previous boundary adds **one day** when the given date falls before the anchor day of
  its own month. For a monthly level anchored on the first and a date of the sixteenth of February,
  it yields the second of February, not the first. That one-day shift is what makes the period
  proration of an out-of-phase first period slightly shorter than a whole month, and it must not be
  smoothed away.

### 4.1 Boundary examples

For a monthly level anchored on the first day of the month:

| Given date | Next boundary | Previous boundary |
|---|---|---|
| 1 January 2024 | 1 February 2024 | 1 January 2024 |
| 16 January 2024 | 1 February 2024 | 1 January 2024 |
| 29 February 2024 | 1 March 2024 | 1 February 2024 |

For a monthly level anchored on the fifteenth:

| Given date | Next boundary | Previous boundary |
|---|---|---|
| 10 January 2024 | 15 January 2024 | 16 December 2023 |
| 15 January 2024 | 15 February 2024 | 15 January 2024 |
| 31 January 2024 | 15 February 2024 | 15 January 2024 |

For a weekly level anchored on Monday:

| Given date | Next boundary | Previous boundary |
|---|---|---|
| Monday 1 January 2024 | Monday 8 January 2024 | Monday 1 January 2024 |
| Thursday 4 January 2024 | Monday 8 January 2024 | Monday 1 January 2024 |

Across every frequency:

| Frequency and settings | Date | Next boundary | Previous boundary |
|---|---|---|---|
| Daily | 1 September | 2 September | 1 September |
| Weekly, Monday | Wednesday 1 September | Monday 6 September | Monday 30 August |
| Twice a month, days 1 and 15 | 1 September | 15 September | 1 September |
| Twice a month, days 1 and 15 | 20 September | 1 October | 15 September |
| Monthly, first day 1 | 16 September | 1 October | 1 September |
| Monthly, first day 20 | 15 March | 20 March | 21 February |
| Twice a year, 1 January and 1 July | 2 September 2021 | 1 January 2022 | 1 July 2021 |
| Yearly, 1 January | 2 September 2021 | 1 January 2022 | 1 January 2021 |

Replacing the day of a month is always clamped to the length of that month: asking for the
thirty-first of February yields the twenty-eighth, or the twenty-ninth in a leap year.

---

## 5. The carry-over cut-off

**Input**: an allocation and a reference date. **Output**: the carry-over cut-off on or after that
reference date.

```formula
cut-off = the first of January of the reference date's year                                   , anchor "At the start of the year"
cut-off = the month and day of the allocation's validity start date, in the reference date's year , anchor "At the allocation date"
cut-off = the plan's carry-over month, and the smaller of the plan's carry-over day and the
          length of that month, in the reference date's year                                  , anchor "Custom date"
```

then

```formula
cut-off = cut-off + 1 year , when the reference date is strictly after the cut-off
```

The comparison is **strict**: a reference date equal to the cut-off keeps that year's cut-off. A
plan anchored at the start of the year, asked about the fifth of March 2024, yields the first of
January 2025; asked about the first of January 2024, it yields the first of January 2024.

---

## 6. The grant of one period

### 6.1 The raw grant

**Input**: the level, the period start, the call start, the period end and the call end.
**Output**: an amount expressed in days.

1. **Proration against worked time.** When the level's frequency is `hourly` or `worked_hours`,
   **or** the plan is based on worked time:

   ```formula
   amount = worked time factor × the level's rate
   ```

   with the factor computed as in [section 6.2](#62-the-worked-time-factor). Otherwise:

   ```formula
   amount = the level's rate
   ```

2. **Unit conversion.** When the level's rate unit is `hour`:

   ```formula
   amount = amount ÷ hours per day( the employee , the allocation's validity start date )
   ```

   Everything downstream of this point is expressed in **days**, whatever the level's rate unit.

3. **Proration against a partial period.** When the call window differs from the period — either
   boundary differing — **and** the plan is **not** based on worked time:

   ```formula
   period days = ( the period end − the period start ) in whole days
   call days   = ( the call end   − the call start )   in whole days
   factor      = minimum( 1 , call days ÷ period days ) , when the period days are not zero
   factor      = 1                                      , otherwise
   amount      = amount × factor
   ```

   This handles the first, out-of-phase period: an allocation starting in the middle of a month
   only earns the share of the month it covers. It is switched off when the plan already prorates
   on worked time, because that proration already accounts for the shorter window.

### 6.2 The worked time factor

**Input**: the level, the period start, the call start, the period end and the call end.

1. Resolve the employee's time zone from the Employee Version in force on the call start, falling
   back to coordinated universal time, and build the call window from the call start at local
   midnight to the call end at local midnight.
2. Inside the call window measure:

   ```formula
   eligible absence hours = the employee's absence hours whose Working Time Exclusion has a time type
                            of `leave` and carries the accrual-eligibility flag
   working hours          = the employee's working hours on their own schedule
   worked                 = working hours + eligible absence hours
   ```

3. When the call window differs from the period, repeat the measurement over the **period** window,
   resolving the time zone from the Employee Version in force on the period start, to obtain the
   *planned worked* hours; otherwise the planned worked hours equal the worked hours.
4. Over whichever window was measured last — the period window when the two differ, the call window
   otherwise — measure:

   ```formula
   ineligible absence hours = the absence hours whose exclusion has a time type of `leave` and does
                              not carry the accrual-eligibility flag
   ```

5. Produce the factor:

   - when the level's frequency is `hourly`:

     ```formula
     factor = planned worked                             , when the plan is based on worked time
     factor = planned worked + ineligible absence hours  , otherwise
     ```

     so an hourly level's rate is a rate **per hour**, multiplied by a number of hours: a level
     granting 0.1 day per hour on a forty-hour week grants four days;

   - otherwise:

     ```formula
     factor = worked ÷ ( ineligible absence hours + planned worked ) , when the divisor is not zero
     factor = 0                                                      , otherwise
     ```

With the attendance companion package installed and a frequency of `worked_hours`, the factor is
replaced by the total worked hours taken from the attendance records: every attendance of the
employee overlapping the call window contributes the part of its duration that falls inside it.

Two consequences are worth stating. First, when the window was widened to the whole period, the
ineligible absence hours are measured over the whole period, not over the call window. Second, for
an hourly frequency the factor is a number of hours, not a ratio.

**Worked example.** A plan based on worked time with a monthly milestone granting four days. In a
month carrying twenty-two working days of eight hours, the employee was absent for five of them on
a type counted as Absence and not eligible for the accrual rate.

```formula
working hours          = 17 × 8 = 136
eligible absence hours = 0
worked                 = 136
planned worked         = 136 , the call window equalling the period
ineligible absence hours = 5 × 8 = 40
factor = 136 ÷ ( 40 + 136 ) = 136 ÷ 176 = 0.772727…
amount = 0.772727… × 4 = 3.090909… days , which the two-decimal display prints as 3.09
```

### 6.3 Adding the grant to the allocation

**Input**: the level, the running cap expressed in days, the consumed amount, the period start and
the period end. **Effect**: the gross balance and the yearly counter both grow.

1. Determine the call window:

   ```formula
   call start = the start date supplied by the caller, defaulting to the allocation's last call
   call end   = the end date supplied by the caller, defaulting to the allocation's next call
   ```

   A period start is computed for a level transition but is **not** applied: the call window always
   begins at the last call date.

2. Compute the raw amount of [section 6.1](#61-the-raw-grant) over that window.

3. **Apply the yearly cap**, when the level caps the yearly accrued time:

   ```formula
   yearly maximum   = the level's yearly maximum                                                 , rate unit Days
   yearly maximum   = the level's yearly maximum ÷ hours per day( employee , the next call date ) , rate unit Hours
   amount           = minimum( amount , yearly maximum − the allocation's yearly accrued amount )
   ```

4. **Apply the running cap**, when the level caps the running balance:

   ```formula
   capped total = the consumed amount + the running cap in days
   amount       = minimum( amount , capped total − the gross balance )
   ```

   Because the gross balance includes the part already consumed, capping the total at *consumed
   plus the maximum* caps the **available** balance at the maximum:

   ```formula
   ( gross balance − consumed amount ) + amount ≤ the running cap
   ```

   An employee who has taken absence therefore keeps accruing, up to the cap on what is still
   available.

5. Apply:

   ```formula
   gross balance         = gross balance         + amount
   yearly accrued amount = yearly accrued amount + amount
   ```

   The amount may be **negative** when a cap has already been exceeded, for instance after a manual
   increase of the balance; the formulas are applied as written, and the balance decreases. This is
   recorded as rule `TOF-160`, a compatibility finding.

The running cap used here is the level's maximum already converted into days: the raw value for a
day-based level, and the raw value divided by the employee's hours per day at the next call date
for an hour-based level. The consumed amount is the allocation-level taken figure of
[calculations.md, chapter 6](calculations.md#6-the-balance-consumption-algorithm), evaluated at the
allocation's next call date with the future ignored, so that the projection cannot recurse into the
engine; for an hour-based type it is converted into days by dividing by the employee's hours per day
at the next call date, falling back to the validity start date.

---

## 7. The engine

**Input**: a set of allocations, a target date defaulting to today, a *force period* flag and a
*log* flag. **Precondition**: each allocation's cursor is either empty, never having run, or
consistent. **Postcondition**: the balance and the cursor are advanced to the target date.
**Failure**: none; an allocation whose plan is misconfigured or has not started is simply skipped.

The engine is **catch-up capable**: one call advances the balance correctly from wherever the cursor
was last left up to the target date, replaying every boundary in between. It is **idempotent** in
the sense that running it twice with the same target date adds nothing the second time, because the
cursor has moved past every boundary already processed.

### 7.1 Preparation

For each allocation compute the seed of the already-accrued flag:

```formula
seed = the allocation's already-accrued flag
       OR ( the gross balance ≠ 0 AND the plan grants at the start of the period )
```

An allocation created with an opening balance, on a plan that grants at the start of the period, is
therefore treated as if the current period had already been granted, which stops the engine from
granting it twice.

### 7.2 Per-allocation set-up

For each allocation:

1. Skip it when its allocation type is not `accrual`.
2. Skip it when its plan carries no level.
3. Let the *first level* be the first level in sequence order, and

   ```formula
   first level start = the allocation's validity start date + the first level's offset
   ```

4. Write the seed into the allocation's already-accrued flag.
5. **When the cursor has never run**, that is when the next call date is empty:

   5.1. When the target date falls **before** the first level's start, skip this allocation
   entirely: the plan has not started, and the cursor stays empty so that a later run retries.

   5.2. Set

   ```formula
   last call        = the later of the existing last call and the first level start,
                      or the first level start when there is no existing last call
   actual last call = last call
   next call        = the first level's next boundary computed from the last call
   ```

   5.3. Lower the next call to the carry-over cut-off when that is earlier:

   ```formula
   next call = minimum( the carry-over cut-off computed from the next call , next call )
   ```

   5.4. Lower it again to the second level's transition date when the plan carries more than one
   level:

   ```formula
   next call = minimum( the allocation's validity start date + the second level's offset , next call )
   ```

   5.5. Unless logging is suppressed, post an internal note on the allocation reading *"This
   allocation have already ran once, any modification won't be effective to the days allocated to
   the employee. If you need to change the configuration of the allocation, delete and create a new
   one."*

### 7.3 The main loop

Repeat while the next call date is **on or before** the target date.

1. **Read the consumed amount** at the next call date, as defined at the end of
   [section 6.3](#63-adding-the-grant-to-the-allocation).

2. **Select the level** in force at the next call date, per
   [chapter 3](#3-selecting-the-level-in-force-on-a-date). When there is none, leave the loop.

3. **Compute the running cap in days**, when the level caps the running balance; otherwise it stays
   zero.

   ```formula
   running cap = the level's maximum                                               , rate unit Days
   running cap = the level's maximum ÷ hours per day( employee , the next call date ) , rate unit Hours
   ```

4. **Compute the candidate next boundary and the current period.**

   ```formula
   candidate    = the level's next boundary computed from the next call
   period start = the level's previous boundary computed from the last call
   period end   = the level's next boundary computed from the last call
   ```

5. **Shorten the candidate for a level transition.** When the level is not the last and the plan
   transitions immediately, let

   ```formula
   level last date = the allocation's validity start date + the next level's offset
   ```

   and, when the next call is **not** that date, lower the candidate to it when it is earlier. When
   the next call **is** exactly that date, raise the *on level transition* flag instead.

6. **Shorten the candidate for the carry-over cut-off.** Let the cut-off be the carry-over cut-off
   computed from the next call. When the next call is strictly before the cut-off and the cut-off is
   strictly before the candidate, lower the candidate to the cut-off.

7. **Handle the carried-over expiry**, only when the level defines a carried-over validity.

   7.1. Take the stored expiry date. Recompute it when it is empty, when the next call has already
   passed it, or when the carried-over pool is zero:

   ```formula
   expiry date = the cut-off + the validity count in days   , validity unit Days
   expiry date = the cut-off + the validity count in months , validity unit Months
   ```

   and store it.

   7.2. When the next call is strictly before the expiry date and the expiry date is strictly before
   the candidate, lower the candidate to the expiry date.

   7.3. When the next call **is** exactly the expiry date, expire the carried-over pool:

   ```formula
   expiring      = maximum( 0 , the carried-over pool − the consumed amount )
   gross balance = maximum( 0 , gross balance − expiring )
   carried-over pool = 0
   ```

   Only the carried-over entitlement that was **not** used for absence expires.

8. **Decide whether this boundary is an accrual date.**

   ```formula
   accrual date = ( the next call = the period end ) OR ( the next call = the level last date )
   ```

9. **Grant, when the plan grants at the start of the period.** When the allocation is not already
   flagged as accrued, and either the boundary is an accrual date or the *on level transition* flag
   is raised, and the plan grants at the **start**, run the grant of
   [section 6.3](#63-adding-the-grant-to-the-allocation) with a call start of the last call and a
   call end of the next call.

10. **Apply the carry-over policy, when the next call is exactly the cut-off.** Record the cut-off
    as the last executed carry-over date, and, when the level loses unused accruals **or** limits
    the carry-over:

    ```formula
    available        = gross balance − the consumed amount
    allowed          = 0                                                   , action "Lost"
    allowed          = minimum( the carry-over maximum , available )       , carry-over "Up to"
    gross balance    = minimum( gross balance , allowed ) + the consumed amount
    ```

    where the carry-over maximum is the level's maximum, divided by the employee's hours per day at
    the validity start date for an hour-based level. In every case where the next call is the
    cut-off, the carried-over pool then becomes the new gross balance. Adding the consumed amount
    back is essential: the stored balance includes the entitlement already spent, so the truncation
    must preserve it.

11. **Grant, when the plan grants at the end of the period.** The same condition as step 9, for a
    plan granting at the **end**.

12. **Reset the yearly counter at the cut-off.** When the next call is the cut-off, the yearly
    accrued amount becomes zero.

13. **Apply the deferred carry-over for plans granting at the start.** Only when the plan grants at
    the start **and** a carry-over has already been executed:

    13.1. Let the *last cut-off* be the recorded last executed carry-over date, select the level in
    force at that date, and let

    ```formula
    carry-over period start = that level's previous boundary computed from the last cut-off
    carry-over period end   = that level's next boundary computed from the last cut-off
    ```

    13.2. When that level is not the last and the plan transitions immediately, lower the carry-over
    period end to the next level's transition date.

    13.3. When that level's frequency is `hourly`, `daily` or `worked_hours`, set the carry-over
    period end to the last cut-off itself: the carry-over period is a single day.

    13.4. Let

    ```formula
    accrued = ( the allocation is not already flagged as accrued ) AND ( the next call = the period end )
    ```

    — note that this uses the plain period end, **not** the accrual-date flag, so that entitlement
    granted on a level-transition date escapes the carry-over policy.

    13.5. When *accrued* holds, the next call lies in the closed interval from the last cut-off to
    the carry-over period end, the actual last call is **not** the carry-over period start — which
    would mean this has already been applied once — and the carry-over level loses unused accruals
    or limits the carry-over, apply the same truncation as step 10, using the current level's
    carry-over maximum, and record the cut-off as the last executed carry-over date.

14. **Advance the cursor.**

    ```formula
    last call        = the next call , only when the boundary is an accrual date
    actual last call = the next call
    next call        = the candidate
    already accrued  = false
    ```

15. **Honour the force-period flag.** When the flag is set and the new next call has passed the
    target date, clamp the next call to the target date and clear the flag. This is what lets a
    caller ask for a prorated partial period.

### 7.4 Pre-granting the open period

After the loop, and only when the plan grants at the **start** of the period, one extra grant may be
placed in advance so that the balance shown today already contains the current period's entitlement.

1. Build the map of level transition dates to levels.
2. Choose the level: the one whose transition date equals the actual last call; failing that the
   level the loop left; failing that the first level.
3. Let the period start be that level's previous boundary computed from the actual last call.
4. Proceed only when the actual last call is one of the period start, the allocation's validity
   start date, or one of the level transition dates; **or** when the actual last call minus the
   level's carried-over validity offset is one of that same set.
5. Let the period end be that level's next boundary computed from the last call, and, when the plan
   transitions immediately and the chosen level is not the last, lower an end date to the next
   level's transition date; otherwise leave that end date unset.
6. Recompute the running cap in days as in step 3 of the loop, but using the **last call** for the
   hours-per-day conversion, and read the consumed amount again.
7. Run the grant of [section 6.3](#63-adding-the-grant-to-the-allocation) with the call window
   explicitly set to the last call and to that end date, or to the period end when no end date was
   set.
8. Raise the already-accrued flag, so that the next run does not grant the same period twice.

### 7.5 The scheduled run

The daily process named "Accrual Time Off: Updates the number of time off" selects every allocation
whose allocation type is `accrual`, whose state is *Approved*, which carries a plan and an employee,
whose validity end date is empty or strictly after the present instant, and whose next call date is
empty or on or before today at midnight. It runs the engine on that selection with a target date of
today, with logging on and without forcing a period.

### 7.6 Running the engine without committing it

Two callers need the engine without persisting its effect.

- The balance computation, when it needs to know how much an allocation will have accrued by a
  future date. It clones the allocation in memory, runs the engine on the clone with that date as
  the target and with logging off, reads the difference and discards the clone. See
  [chapter 8](#8-projecting-future-accrual-without-committing-it).
- The allocation form, when the user changes the validity start date, the validity end date, the
  plan or the employee on an allocation that is not yet *Approved*. It resets the cursor — the last
  call to the validity start date, the next call cleared, every amount zeroed, the already-accrued
  flag cleared, the expiry date and the carried-over pool cleared — and then runs the engine up to
  the earlier of the validity end date and today, so that the user sees, before saving, what the
  plan would already have granted. This is rule `TOF-068`.

---

## 8. Projecting future accrual without committing it

**Input**: an allocation and a future date. **Output**: the amount the allocation will have gained
between today and that date, in days, or in hours for an hour-based type.

1. The projection is zero when the date is empty or is not strictly after today.
2. The projection is zero unless the allocation carries a plan, its state is *Approved*, its
   allocation type is `accrual`, its validity end date is empty or strictly after the target date,
   and its next call date is empty or on or before the target date.
3. Otherwise clone the allocation in memory, run the engine of [chapter 7](#7-the-engine) on the
   clone with that target date and with logging suppressed, and take the difference of the displayed
   hour amounts for an hour-based type, or of the day amounts otherwise, rounded to two decimal
   places.
4. Discard the clone.

The same mechanism serves the carried-over expiry projection: the clone is advanced to the
evaluation date and its expiry date and carried-over pool are read back, as specified in
[calculations.md, section 13.2](calculations.md#132-the-carried-over-expiry-projection).

Recursion is prevented by two mechanisms: the consumption algorithm passes the set of allocations it
is already simulating down through the calling context, and the projection is skipped entirely when
the consumption algorithm is told to ignore the future.

---

## 9. Worked example one — 1.25 days a month, a running cap of 15, a carry-over limited to 5

### 9.1 The configuration

The plan, named "Seniority":

| Setting | Value |
|---|---|
| Accrued gain time | At the end of the accrual period |
| Based on worked time | no |
| Carry-over time | At the start of the year |
| Carry-over allowed | yes |
| Milestone transition | Immediately, irrelevant with a single milestone |
| Value unit | Days |

Its single milestone:

| Setting | Value |
|---|---|
| Milestone reached | At allocation creation, offset zero |
| Rate | 1.25 Day(s) |
| Frequency | Monthly, anchored on day 1 |
| Cap accrued time | on, 15 days |
| Cap accrued time yearly | off |
| Unused accruals | Carried over |
| Carry-over options | Up to 5 days |
| Carried-over validity | off |

The allocation belongs to an employee named Bob, of the type Paid Time Off whose request unit is
`day`, of kind Accrual Allocation, driven by the plan "Seniority", valid from the first of November
2023 with no end date, with an opening amount of zero, approved on the first of November 2023. The
observation window runs to the thirty-first of December 2024, fourteen months.

### 9.2 Initialisation

```formula
seed of the already-accrued flag = false , the gross balance being zero
first level start = 1 November 2023 + 0 days = 1 November 2023
the target date of the first run is on or after 1 November 2023 , so the plan has started
last call        = 1 November 2023
actual last call = 1 November 2023
next call        = the monthly next boundary from 1 November 2023 = 1 December 2023
the carry-over cut-off from 1 December 2023 is 1 January 2024 , which is later , so the next call stands
```

### 9.3 The trace

| Iteration date | Period closed | Carry-over applied first | Grant | Clipped by the cap | Balance after |
|---|---|---|---|---|---|
| 1 December 2023 | 1 November to 1 December 2023 | no | 1.25 | no | 1.25 |
| 1 January 2024 | 1 December 2023 to 1 January 2024 | yes: available 1.25, limit 5, nothing lost | 1.25 | no | 2.50 |
| 1 February 2024 | 1 January to 1 February | no | 1.25 | no | 3.75 |
| 1 March 2024 | 1 February to 1 March | no | 1.25 | no | 5.00 |
| 1 April 2024 | 1 March to 1 April | no | 1.25 | no | 6.25 |
| 1 May 2024 | 1 April to 1 May | no | 1.25 | no | 7.50 |
| 1 June 2024 | 1 May to 1 June | no | 1.25 | no | 8.75 |
| 1 July 2024 | 1 June to 1 July | no | 1.25 | no | 10.00 |
| 1 August 2024 | 1 July to 1 August | no | 1.25 | no | 11.25 |
| 1 September 2024 | 1 August to 1 September | no | 1.25 | no | 12.50 |
| 1 October 2024 | 1 September to 1 October | no | 1.25 | no | 13.75 |
| 1 November 2024 | 1 October to 1 November | no | 1.25 | no | 15.00 |
| 1 December 2024 | 1 November to 1 December | no | 1.25 requested | yes | 15.00 |

The carry-over of the first of January 2024, which happens **before** the grant of the December
period because the plan grants at the end:

```formula
available     = 1.25 − 0 = 1.25
allowed       = minimum( 5 , 1.25 ) = 1.25
gross balance = minimum( 1.25 , 1.25 ) + 0 = 1.25 , nothing is lost
carried-over pool = 1.25
then the grant of the December period lands : 1.25 , balance 2.50
then the yearly accrued amount is reset to zero
```

The clipping of the first of December 2024:

```formula
consumed amount = 0
running cap     = 15
amount = minimum( 1.25 , ( 0 + 15 ) − 15 ) = minimum( 1.25 , 0 ) = 0
```

**Balance on the thirty-first of December 2024: 15.00 days, all of it available.** Thirteen grants
of 1.25 total 16.25 days, of which 1.25 were refused by the running cap.

### 9.4 What happens at the next cut-off

On the first of January 2025 the engine performs, in order:

```formula
carry-over : available = 15 − 0 = 15 ; allowed = minimum( 5 , 15 ) = 5
             gross balance = minimum( 15 , 5 ) + 0 = 5 ; carried-over pool = 5
grant of the December 2024 period : minimum( 1.25 , ( 0 + 15 ) − 5 ) = 1.25
balance after 1 January 2025 = 6.25
```

Ten days are therefore lost on the first of January 2025.

### 9.5 The same plan with an absence taken

Suppose Bob takes four days of Paid Time Off from the fifteenth to the eighteenth of July 2024,
approved and charged entirely against this allocation.

| Iteration date | Consumed | Cap arithmetic | Balance after | Available after |
|---|---|---|---|---|
| 1 July 2024 | 0 | minimum( 1.25 , ( 0 + 15 ) − 8.75 ) = 1.25 | 10.00 | 10.00 |
| 1 August 2024 | 4 | minimum( 1.25 , ( 4 + 15 ) − 10.00 ) = 1.25 | 11.25 | 7.25 |
| 1 September 2024 | 4 | minimum( 1.25 , 19 − 11.25 ) = 1.25 | 12.50 | 8.50 |
| 1 October 2024 | 4 | minimum( 1.25 , 19 − 12.50 ) = 1.25 | 13.75 | 9.75 |
| 1 November 2024 | 4 | minimum( 1.25 , 19 − 13.75 ) = 1.25 | 15.00 | 11.00 |
| 1 December 2024 | 4 | minimum( 1.25 , 19 − 15.00 ) = 1.25 | 16.25 | 12.25 |

The absence therefore unlocks further accrual: the gross amount rises to 16.25 while the available
balance stays at or below the fifteen-day cap. On the first of January 2025:

```formula
available     = 16.25 − 4 = 12.25
allowed       = minimum( 5 , 12.25 ) = 5
gross balance = minimum( 16.25 , 5 ) + 4 = 9
grant of the December period = minimum( 1.25 , ( 4 + 15 ) − 9 ) = 1.25
gross balance = 10.25 , available = 10.25 − 4 = 6.25
```

---

## 10. Worked example two — a daily plan granting at the start

This example exercises the pre-grant mechanism, the running cap and the carry-over on the same
cut-off.

### 10.1 The configuration

| Plan setting | Value |
|---|---|
| Accrued gain time | At the start of the accrual period |
| Based on worked time | no, forced |
| Carry-over time | At the start of the year |
| Carry-over allowed | yes |

| Milestone setting | Value |
|---|---|
| Milestone reached | After 1 Day |
| Rate | 1 Day |
| Frequency | Daily |
| Cap accrued time | on, 25 days |
| Unused accruals | Carried over |
| Carry-over options | Up to 15 days |

The allocation is valid from the fifteenth of December 2021, carries an opening amount of ten days,
is of kind Accrual Allocation and was approved the same day. The engine is run with a target date of
the first of January 2022.

### 10.2 Initialisation

```formula
seed of the already-accrued flag = true , the opening amount not being zero and the plan granting at the start
first level start = 15 December 2021 + 1 day = 16 December 2021
last call = actual last call = 16 December 2021
next call = the daily next boundary from 16 December 2021 = 17 December 2021
the carry-over cut-off from 17 December 2021 is 1 January 2022 , which is later , so the next call stands
```

### 10.3 The trace

| Iteration date | Already-accrued flag on entry | Grant | Balance after |
|---|---|---|---|
| 17 December 2021 | true, so the grant is skipped | none | 10 |
| 18 December 2021 | false | 1 | 11 |
| 19 to 31 December 2021 | false | 1 each, thirteen grants | 24 |
| 1 January 2022 | false | the start-of-period grant: minimum( 1 , ( 0 + 25 ) − 24 ) = 1 | 25 |

On the first of January 2022, after the start-of-period grant, the cut-off fires:

```formula
available     = 25 − 0 = 25
allowed       = minimum( 15 , 25 ) = 15
gross balance = minimum( 25 , 15 ) + 0 = 15
carried-over pool = 15
```

Step 13 then runs, because the plan grants at the start and a carry-over has now been executed. The
carry-over level is the only level; its previous boundary from the first of January 2022 is the
first of January and its next boundary is the second, but because the frequency is Daily the
carry-over period end is pulled back to the cut-off date itself. The condition holds, the same
arithmetic is applied again, and because minimum( 15 , minimum( 15 , 15 ) ) plus zero is fifteen the
balance does not change.

Finally the loop ends and the pre-grant of [section 7.4](#74-pre-granting-the-open-period) runs:

```formula
actual last call = 1 January 2022 = the daily previous boundary from 1 January 2022 , so the block proceeds
period end = the daily next boundary from the last call of 1 January 2022 = 2 January 2022
amount = minimum( 1 , ( 0 + 25 ) − 15 ) = 1
balance = 16 ; the already-accrued flag is raised
```

**Balance on the first of January 2022: sixteen days.** Fifteen of them are the carried-over pool and
one is the day already granted for the period that opened on the first of January.

---

## 11. Worked example three — two milestones and the two transition modes

The plan, named "Career": carry-over allowed, grant at the end of the period, carry-over at the
allocation date.

| Milestone | Reached | Rate | Frequency |
|---|---|---|---|
| First | At allocation creation | 1 day | Monthly on day 1 |
| Second | After 13 Months | 2 days | Monthly on day 1 |

The allocation is valid from the first of January 2023 with an opening amount of zero, so the second
milestone's transition date is the first of February 2024.

**Transition mode "Immediately".** The loop reaches the first of February 2024 with a level last date
of the first of February 2024 and a next call equal to it, which raises the *on level transition*
flag. The level in force at that date is still the **first** one, because the selection of
[chapter 3](#3-selecting-the-level-in-force-on-a-date) compares strictly, so the grant of the first
of February is **one day**. From the first of March 2024 onwards the level in force is the second one
and every grant is **two days**.

**Transition mode "After this accrual's period".** The selection adds step 5. At the first of
February 2024 the retained level is the first one anyway. At the first of March 2024 the retained
level is the second one, at index one, so step 5 compares the second level's next boundary from the
transition date, the first of March 2024, with the first level's next boundary from the same date,
also the first of March 2024; the comparison is strict, so the second level is kept and the grant of
the first of March is two days. The two modes therefore coincide here, because both levels use the
same frequency and anchor.

They diverge as soon as the second level uses a different frequency. Replace the second milestone by
one that is **Yearly on the first of January**, still reached after thirteen months. At the first of
March 2024 the comparison becomes the second level's next boundary from the first of February 2024,
which is the first of January 2025, against the first level's next boundary from the same date, the
first of March 2024. The second is earlier, so the **first** level is kept until its own period
closes on the first of March 2024, and only afterwards does the second level take over.

---

## 12. Worked example four — a carry-over with a validity window

The plan grants at the end of the period, anchors its carry-over at the allocation date, carries a
monthly milestone granting one day, an action of "Carried over" with no limit, and a carried-over
validity of three Months. The allocation is valid from the fifteenth of January 2023 with an opening
amount of zero, so the cut-off is the fifteenth of January of each year.

```formula
15 February 2023 through 15 December 2023 : eleven grants of 1 day , balance 11
15 January 2024 : the cut-off fires. The action is "Carried over" with no limit, so neither the
                  "Lost" branch nor the limited branch applies and the balance is untouched.
                  The carried-over pool becomes the balance, which is 11 before the January grant,
                  and the expiry date is computed as 15 January 2024 + 3 months = 15 April 2024.
                  The January period then grants 1 day , balance 12.
15 February 2024 and 15 March 2024 : one day each , balance 14
15 April 2024 : the next call equals the expiry date, so
                expiring      = maximum( 0 , 11 − the consumed amount )
                gross balance = maximum( 0 , 14 − expiring )
                the carried-over pool is reset to zero
```

With no absence taken, the expiring amount is eleven and the balance drops to **three**, which is
exactly the three days accrued since the cut-off. With four days consumed since the start, the
expiring amount is maximum( 0 , 11 − 4 ) = 7 and the balance drops to 14 − 7 = 7, of which four are
already consumed and three are available.

The expiry date is recomputed at the next cut-off, because the carried-over pool has been reset to
zero, which satisfies the third condition of step 7.1 of the loop.

---

## 13. Worked example five — fourteen months at one and a half days a month

### 13.1 The configuration

| Setting | Value |
|---|---|
| Plan — carry-over time | At the start of the year, so the cut-off is the first of January |
| Plan — carry-over allowed | yes |
| Plan — accrued gain time | At the end of the accrual period |
| Plan — based on worked time | no |
| Plan — milestone transition | Immediately, irrelevant with one milestone |
| Milestone — reached | At allocation creation, start count zero, unit Days |
| Milestone — rate | **1.5**, unit Days |
| Milestone — frequency | Monthly, first day **1** |
| Milestone — cap accrued time | yes, maximum **20** days |
| Milestone — unused accruals | Carried over |
| Milestone — carry-over options | Up to **5** days |
| Milestone — carried-over validity | off |
| Allocation — validity start | 1 January 2024 |
| Allocation — kind and state | Accrual Allocation, *Approved*, opening balance zero |
| Employee | the standard schedule, eight hours a day, **no absence taken** in the period |

### 13.2 Priming the cursor

At creation, on the first of January 2024:

```formula
first level start = 1 January 2024 + 0 days = 1 January 2024
no level has a transition date strictly before today, but the first level's transition date equals
   today, so the first level is taken, at index zero
last call        = the later of the monthly previous boundary from 1 January 2024 and 1 January 2024
                 = the later of 1 January 2024 and 1 January 2024 = 1 January 2024
actual last call = 1 January 2024
next call        = the monthly next boundary from 1 January 2024 = 1 February 2024
```

The seed of the already-accrued flag is false, because the plan grants at the **end**.

### 13.3 The fourteen iterations

The engine is run with a target date of the first of March 2025. Every iteration follows the same
shape; the interesting one is the twelfth. For a generic iteration at a next call date **B**:

```formula
consumed amount = 0
level           = the single level
running cap     = 20
candidate       = the monthly next boundary from B = the first of the following month
period start    = the monthly previous boundary from the last call = the last call
period end      = the monthly next boundary from the last call     = B
cut-off         = the carry-over cut-off computed from B
accrual date    = ( B = the period end ) = true
raw amount      = 1.5 , no hourly frequency and not based on worked time
rate unit Days  , so no unit conversion
call window     = ( the last call , the next call ) = ( the period start , the period end ) , so the period factor is 1
yearly cap off
running cap     : capped total = 0 + 20 = 20 ; amount = minimum( 1.5 , 20 − the balance )
```

| # | Next call date | Cut-off computed | Balance before | Carry-over applied | Grant | Balance after | Yearly counter after |
|---|---|---|---|---|---|---|---|
| 1 | 1 February 2024 | 1 January 2025 | 0 | no | +1.5 | **1.5** | 1.5 |
| 2 | 1 March 2024 | 1 January 2025 | 1.5 | no | +1.5 | **3.0** | 3.0 |
| 3 | 1 April 2024 | 1 January 2025 | 3.0 | no | +1.5 | **4.5** | 4.5 |
| 4 | 1 May 2024 | 1 January 2025 | 4.5 | no | +1.5 | **6.0** | 6.0 |
| 5 | 1 June 2024 | 1 January 2025 | 6.0 | no | +1.5 | **7.5** | 7.5 |
| 6 | 1 July 2024 | 1 January 2025 | 7.5 | no | +1.5 | **9.0** | 9.0 |
| 7 | 1 August 2024 | 1 January 2025 | 9.0 | no | +1.5 | **10.5** | 10.5 |
| 8 | 1 September 2024 | 1 January 2025 | 10.5 | no | +1.5 | **12.0** | 12.0 |
| 9 | 1 October 2024 | 1 January 2025 | 12.0 | no | +1.5 | **13.5** | 13.5 |
| 10 | 1 November 2024 | 1 January 2025 | 13.5 | no | +1.5 | **15.0** | 15.0 |
| 11 | 1 December 2024 | 1 January 2025 | 15.0 | no | +1.5 | **16.5** | 16.5 |
| 12 | **1 January 2025** | **1 January 2025** | 16.5 | **yes, truncated to 5** | +1.5 | **6.5** | reset to 0 |
| 13 | 1 February 2025 | 1 January 2026 | 6.5 | no | +1.5 | **8.0** | 1.5 |
| 14 | 1 March 2025 | 1 January 2026 | 8.0 | no | +1.5 | **9.5** | 3.0 |

### 13.4 The twelfth iteration in detail

```formula
next call    = 1 January 2025
last call    = 1 December 2024
period start = the monthly previous boundary from 1 December 2024 = 1 December 2024
period end   = the monthly next boundary from 1 December 2024     = 1 January 2025
candidate    = the monthly next boundary from 1 January 2025      = 1 February 2025
cut-off      = the first of January of 2025 = 1 January 2025
               , the comparison "1 January 2025 strictly after 1 January 2025" being false, no year is added
step 6  : the next call is not strictly before the cut-off , so the candidate is unchanged
step 8  : accrual date = ( 1 January 2025 = 1 January 2025 ) = true
step 9  : the plan grants at the end, so nothing happens here
step 10 : the next call equals the cut-off , so the carry-over policy applies
          the last executed carry-over date becomes 1 January 2025
          the action is "Carried over" and the carry-over is limited , so the branch runs
          available     = 16.5 − 0 = 16.5
          allowed       = minimum( 5 , 16.5 ) = 5
          gross balance = minimum( 16.5 , 5 ) + 0 = 5
          carried-over pool = 5
step 11 : the plan grants at the end and the boundary is an accrual date , so the grant runs
          raw amount  = 1.5
          running cap : capped total = 0 + 20 = 20 ; amount = minimum( 1.5 , 20 − 5 ) = 1.5
          gross balance         = 5 + 1.5 = 6.5
          yearly accrued amount = 16.5 + 1.5 = 18.0
step 12 : the next call equals the cut-off , so the yearly accrued amount becomes 0
step 14 : last call = 1 January 2025 ; actual last call = 1 January 2025 ; next call = 1 February 2025
```

**Result after fourteen months: a balance of 9.5 days.** Twenty-one days were accrued in total,
fourteen grants of one and a half; eleven and a half were lost at the carry-over cut-off.

### 13.5 Where the running cap bites

Continuing the same allocation past the fourteenth iteration, with no absence taken:

| Next call date | Balance before | Grant | Balance after |
|---|---|---|---|
| 1 April 2025 | 9.5 | +1.5 | 11.0 |
| 1 May 2025 | 11.0 | +1.5 | 12.5 |
| 1 June 2025 | 12.5 | +1.5 | 14.0 |
| 1 July 2025 | 14.0 | +1.5 | 15.5 |
| 1 August 2025 | 15.5 | +1.5 | 17.0 |
| 1 September 2025 | 17.0 | +1.5 | 18.5 |
| 1 October 2025 | 18.5 | +1.5 | **20.0** |
| 1 November 2025 | 20.0 | minimum( 1.5 , 20 − 20 ) = **+0** | 20.0 |
| 1 December 2025 | 20.0 | +0 | 20.0 |
| 1 January 2026 | 20.0 | truncated to 5, then +1.5 | 6.5 |

### 13.6 The same plan with absence taken

Suppose the employee takes **four days** of approved absence in June 2024, charged to this
allocation. The stored balance is unaffected by taking absence, because it is the gross accrued
figure; what changes is the consumed amount read at each boundary. At the twelfth iteration:

```formula
consumed amount = 4
available       = 16.5 − 4 = 12.5
allowed         = minimum( 5 , 12.5 ) = 5
gross balance   = minimum( 16.5 , 5 ) + 4 = 9
carried-over pool = 9
then the grant : capped total = 4 + 20 = 24 ; amount = minimum( 1.5 , 24 − 9 ) = 1.5
gross balance  = 10.5
```

and the balance the employee can still use is

```formula
available = gross balance − consumed amount = 10.5 − 4 = 6.5
```

— exactly the five days carried over plus the one and a half granted on the first of January. Adding
the consumed amount back inside the truncation is what makes the carry-over cap apply to the
**unused** balance rather than to the gross figure.

### 13.7 The same plan granting at the start of the period

If the plan is switched to grant at the start of the accrual period, three things change:

1. the grant of step 9 fires instead of the grant of step 11, so it happens **before** the carry-over
   truncation of step 10 rather than after;
2. the deferred carry-over of step 13 catches the entitlement granted at the start of a period that
   spans the cut-off;
3. after the loop, the pre-grant of [section 7.4](#74-pre-granting-the-open-period) grants the
   current period in advance and raises the already-accrued flag.

With the same numbers, the balance on the first of January 2025 becomes: sixteen and a half carried
into the iteration, plus one and a half granted at the start of the January period by step 9, giving
eighteen; then truncated by step 10 to five; then the pre-grant adds the February period's one and a
half at the end of the run, giving six and a half.

---

## 14. Interaction with the balance

An accrual allocation contributes to the balance exactly like a regular allocation, with one
addition: when the balance is asked for a **future** date, the engine is simulated on a clone up to
that date and the difference is published as the allocation's accrual bonus, which is added to its
maximum allowed. This is what lets an employee see, on the dashboard, how much entitlement they will
hold on a chosen future date.

Absences that start after the evaluation date and are funded by an accrual allocation are not
charged at that date at all: they are set aside and compared, as a block, against the balance the
accrual will have reached on the latest of their start dates. The resulting shortfall is published
separately as the exceeding duration. The rule is `TOF-058` and the arithmetic is in
[calculations.md, section 6.2](calculations.md#62-steps), step 7.

---

## 15. Interaction with the daily invalid-absence run

Because an accrual balance grows over time, an absence booked far in the future may be affordable
when it is booked and unaffordable later, for instance because the employee took other absences in
between, or because a level transition reduced the rate, or because a carry-over cut-off truncated
the balance. The daily process named "Time Off: Cancel invalid leaves" looks thirty-one days ahead
every day and cancels, from the furthest to the nearest, the absences that the accrual balance can no
longer cover. The rule is `TOF-140` and the procedure is
[workflows.md, chapter 13](workflows.md#13-the-daily-invalid-absence-run).

---

## 16. Reconciliation notes

1. **Where this file came from.** One of the two source drafts of this folder carried the accrual
   machinery inside its calculations document as six chapters; the other carried it in a file of its
   own. This file merges both: the ordered engine and the configuration surface come from the
   dedicated file, the step-by-step detail of the deferred carry-over and of the pre-grant from the
   calculations chapters, and every worked example of both drafts is retained —
   [chapters 9 to 12](#9-worked-example-one--125-days-a-month-a-running-cap-of-15-a-carry-over-limited-to-5)
   from one draft and [chapter 13](#13-worked-example-five--fourteen-months-at-one-and-a-half-days-a-month)
   from the other.
2. **The name of the yearly anchor.** One draft called it the carry-over anchor, the other the
   carry-over cut-off. Both names are recorded in [chapter 1](#1-vocabulary) and the cut-off is used
   throughout, because it is the word the closest-expiry computation of
   [calculations.md, chapter 9](calculations.md#9-the-closest-expiring-entitlement) uses.
3. **The consumed amount read by the loop.** One draft said it is read at the next call date with the
   future ignored, the other that it is read at the next call date; the first is the complete
   statement and is kept, because it is what prevents the projection from recursing into the engine.
4. **The monthly previous boundary.** Both drafts recorded the extra day; one gave the worked example
   with an anchor on the twentieth and the other with an anchor on the first. Both examples are kept
   in [section 4.1](#41-boundary-examples).
5. **The level transition comparison.** One draft stated the comparison of step 5 of the level
   selection as strict, the other did not qualify it. The source is strict, and
   [chapter 11](#11-worked-example-three--two-milestones-and-the-two-transition-modes) shows the case
   where the difference is observable.
6. **The pre-grant precondition.** Only one draft recorded the alternative condition on the actual
   last call minus the carried-over validity offset. It is retained in
   [section 7.4](#74-pre-granting-the-open-period), step 4.
