# Work Entries — Acceptance Criteria

Numbered Given, When and Then scenarios with concrete starting records, concrete inputs and exact
resulting records. A rebuild is behaviourally equivalent to the specified system when every one of
them passes.

Six of them are mandatory: a rebuild that fails any of scenarios 1 to 6 is not equivalent in any
useful sense. They are named as such in [README.md, chapter 6](README.md#6-mandatory-scenarios).

---

## 1. The standard fixture

Unless a scenario says otherwise, the following records exist before it starts.

| Record | Value |
|---|---|
| Company | one company, country Belgium, currency the euro, its own working schedule Monday to Friday 08:00–12:00 and 13:00–17:00, zone Europe/Brussels |
| Working schedule "Standard 40 hours" | zone Europe/Brussels, eight hours per day; ten lines, Monday to Friday, a morning line 08:00–12:00 and an afternoon line 13:00–17:00, each carrying the shipped ordinary-attendance kind, payroll code `WORK100` |
| Employee | one employee, resource zone Europe/Brussels, department "Research and Development" |
| Employee Version | version date 1 January 2024; contract start date 1 January 2024; no contract end date; working schedule "Standard 40 hours"; generation source `calendar`; both markers equal, that is the sentinel |
| Work entry kinds | the nine universal shipped kinds |
| Acting user | a Human Resources Administrator of that company |

Dates are given as calendar dates in the schedule's zone. September 2025 begins on a Monday; 1, 2, 3,
4 and 5 September are Monday to Friday; 6 and 7 September are the weekend.

---

## 2. Generation, the ordinary paths

### Scenario 1: Generating one week for a five-day, eight-hour employee

**Given** the standard fixture, and no work entry exists.

**When** generation is requested for the period 1 September 2025 to 7 September 2025, not forced.

**Then**

1. Exactly five work entries exist, dated 1, 2, 3, 4 and 5 September 2025.
2. Each carries the kind "Attendance", payroll code `WORK100`, a duration of 8.000 hours, the state
   `draft`, the archived flag true, the employee, the version, the company and a pay rate of 1.0.
3. Each carries the description "Attendance: " followed by the employee's name.
4. No entry exists for 6 or 7 September 2025.
5. The total of the five durations is 40.000 hours.
6. The version's generated-from marker reads 2025-08-31 22:00:00 and its generated-to marker reads
   2025-09-07 21:59:59.999999, both on the universal time scale.
7. The version's last generation date reads the date the run happened.

### Scenario 2: A validated absence day replaces the generated entry

**Given** the result of scenario 1, and an absence kind "Paid Time Off" whose work entry kind is the
shipped "Paid Time Off", payroll code `LEAVE120`.

**When** an absence request for the whole of Wednesday 3 September 2025 is validated.

**Then**

1. A work entry dated 3 September 2025 exists carrying the kind "Paid Time Off", payroll code
   `LEAVE120`, a duration of 8.000 hours, the state `draft` and a link to the absence request.
2. The work entry dated 3 September 2025 that carried `WORK100` has its archived flag false and its
   state `cancelled`.
3. The entries of 1, 2, 4 and 5 September are unchanged: still `WORK100`, still 8.000 hours, still
   `draft`.
4. The total of the durations of the live entries of the week is still 40.000 hours.
5. No entry is in the conflict state.

### Scenario 3: An overlapping manual entry raises a conflict

**Given** the result of scenario 1.

**When** a work entry is created by hand for the same employee on 1 September 2025, with the kind
"Overtime Hours", payroll code `OVERTIME`, and a duration of 17.000 hours.

**Then**

1. The day's total is 8.000 + 17.000 = 25.000 hours, which is greater than twenty-four.
2. **Both** entries of 1 September 2025 are in the state `conflict` and both carry the stored conflict
   indicator true.
3. The entries of 2, 3, 4 and 5 September are untouched and remain `draft`.
4. Invoking the validation operation over the five days of the week returns false and writes no
   `validated` state anywhere.
5. When the manual entry's duration is then changed to 16.000 hours, the day's total becomes 24.000,
   the reset pass returns both entries to `draft`, the re-check finds nothing, and both entries end in
   the state `draft`.

### Scenario 4: Regeneration after a schedule change

**Given** an employee whose version starts on 1 January 2025 with a working schedule of forty hours a
week, eight hours a day, Monday to Friday 07:00–11:00 and 13:00–17:00, zone Europe/Brussels; and
generation has been run for 1 January 2025 to 31 January 2025, producing twenty-three entries
totalling 184.000 hours.

**When** a second Employee Version is created, dated 10 January 2025, naming a working schedule of
thirty-five hours a week, seven hours a day, Monday to Friday 07:00–11:00 and 13:00–16:00; and
generation is requested again for 1 January 2025 to 31 January 2025.

**Then**

1. Twenty-three entries exist for the month.
2. Their durations total 168.000 hours.
3. Seven of them, dated 1, 2, 3, 6, 7, 8 and 9 January, carry 8.000 hours each and the **first**
   version.
4. Sixteen of them, dated 10, 13, 14, 15, 16, 17, 20, 21, 22, 23, 24, 27, 28, 29, 30 and 31 January,
   carry 7.000 hours each and the **second** version.
5. The entries produced under the first version for the days from 10 January onwards are archived and
   cancelled; none of them is deleted.
6. No entry is in the conflict state.

### Scenario 5: A version ending on a Wednesday

**Given** the standard fixture with a contract end date of Wednesday 17 September 2025, and generation
already run for 1 September 2025 to 30 September 2025, so that twenty-two entries exist and the
generated-to marker reads 2025-09-30 21:59:59.999999.

**When** generation is requested again for 1 September 2025 to 30 September 2025, not forced.

**Then**

1. The version's own stop is 2025-09-17 21:59:59.999999 on the universal scale, which is earlier than
   the window stop, and the two markers differ, so the entries beyond it are retired.
2. The nine entries dated 18, 19, 22, 23, 24, 25, 26, 29 and 30 September, totalling 72.000 hours, have
   their archived flag false and their state `cancelled`.
3. The thirteen entries dated 1 to 17 September are untouched and remain `draft`.
4. No new entry is produced: the generated-to marker is not earlier than the clamped generation stop.
5. The generated-to marker still reads 2025-09-30 21:59:59.999999. It now overstates the coverage;
   this is the **compatibility finding** recorded in
   [calculations.md, section 10.5](calculations.md#105-worked-example-a-contract-ending-on-a-wednesday).

**And when** instead the contract end date of 17 September 2025 is **written onto the version** while
those twenty-two entries exist,

**then** the out-of-period removal runs: the nine entries dated after 17 September are **deleted**, and
the generated-to marker is pulled back to 2025-09-17 23:59:59.999999.

### Scenario 6: The half-day rounding rule

**Given** a working schedule prescribing eight hours a day.

**When** the day contribution of each of the following is computed:

**Then**

| Line or interval | Arithmetic | Day contribution |
|---|---|---|
| A morning line 08:00–12:00 | span 4.00 hours; three quarters of the daily hours is 8 × 3 ÷ 4 = 6.00; 4.00 ≤ 6.00 | 0.5 |
| An afternoon line 13:00–17:00 | span 4.00; 4.00 ≤ 6.00 | 0.5 |
| A morning line 08:00–15:00 | span 7.00; 7.00 > 6.00 | 1 |
| A morning line 08:00–14:00 | span 6.00; 6.00 ≤ 6.00, the comparison is not strict | 0.5 |
| A full-day line | by definition | 1 |
| A break line | by definition | 0 |
| An interval 09:00–11:00 inside the 08:00–12:00 line | 0.5 × 2.00 ÷ 4.00 | 0.250 |
| An interval 08:00–10:20 inside the 08:00–15:00 line | 1 × 2.333333… ÷ 7.00, rounded to a thousandth | 0.333 |
| A six-hour interval on a flexible-hours schedule of eight hours a day | 6.00 ÷ 8.00 | 0.750 |

And the day total of the morning line plus the afternoon line is 0.5 + 0.5 = 1.000.

### Scenario 7: Generation is idempotent

**Given** the result of scenario 1.

**When** generation is requested again for 1 September 2025 to 7 September 2025, not forced.

**Then**

1. No entry is created and none is changed; five entries still exist.
2. Both markers are unchanged.
3. The last generation date is rewritten to today, because it records attempts.

### Scenario 8: An absence measured in hours splits the day

**Given** the standard fixture, and an absence kind whose work entry kind is the shipped "Sick Time
Off", payroll code `LEAVE110`.

**When** generation is run for 1 September 2025 to 7 September 2025 and an absence from 10:00 to 12:00
on Wednesday 3 September is validated.

**Then**

1. Wednesday 3 September carries exactly two live entries: one of kind `WORK100` with a duration of
   6.000 hours, and one of kind `LEAVE110` with a duration of 2.000 hours and a link to the request.
2. The original eight-hour entry of that day is archived and cancelled.
3. The day totals 8.000 hours and is not in conflict.
4. The other four days are untouched.

### Scenario 9: A half-day absence spanning two days

**Given** the standard fixture, with generation already run for November 2025, so that Thursday 27 and
Friday 28 November each carry one entry of 8.000 hours of kind `WORK100`.

**When** an absence of a half-day kind is validated, requested from the morning of 27 November to the
morning of 28 November.

**Then**

1. Three live entries exist across the two days.
2. 27 November carries one entry of the absence's work entry kind with a duration of 8.000 hours; its
   former attendance entry is archived.
3. 28 November carries one entry of the absence's work entry kind with a duration of 4.000 hours and
   one entry of `WORK100` with a duration of 4.000 hours.
4. Each day totals 8.000 hours.

### Scenario 10: A public holiday outranks an individual absence

**Given** the standard fixture; a work entry kind "Public Holiday", payroll code `PUBHOL`, carrying the
absence flag; a company closure on the schedule covering 6 and 7 February 2023 with that kind and no
resource; and an absence kind whose work entry kind is the shipped "Paid Time Off", payroll code
`LEAVE120`.

**When** an absence request covering 3 February to 9 February 2023 is validated and generation is
forced for February 2023.

**Then**

1. 6 and 7 February carry entries of kind `PUBHOL` with **no** absence link, because a company closure
   is not backed by a request.
2. 3, 8 and 9 February carry entries of kind `LEAVE120`, each linked to the absence request.
3. 4 and 5 February are the weekend and carry no entry.
4. Every one of those entries carries 8.000 hours.

### Scenario 11: A worked absence counts as worked time

**Given** the standard fixture and a work entry kind "Training", payroll code `TRAIN`, **not** carrying
the absence flag.

**When** a Working Time Exclusion is created on the schedule for Monday 8 September 2025 from 08:00 to
17:00, with the time type `other` "Other" and that kind, and generation is forced for that day.

**Then**

1. One entry exists for 8 September 2025, of kind `TRAIN`, with a duration of 8.000 hours.
2. The duration was measured by the clock spans of the two surviving attendance intervals, 4.000 and
   4.000, and not by the theoretical schedule, because the kind does not carry the absence flag.
3. No entry of kind `WORK100` exists for that day.

### Scenario 12: A fully flexible version produces whole days

**Given** an employee whose version names **no** working schedule, whose contract starts on 1
September 2025 and ends on 15 September 2025, zone Europe/Brussels.

**When** generation is requested for 1 September 2025 to 30 September 2025.

**Then**

1. Fifteen entries exist, dated 1 to 15 September inclusive, including the weekends of 6, 7, 13 and 14
   September.
2. Each carries a duration of 24.000 hours.
3. Each carries the shipped ordinary-attendance kind, because the single whole-window interval carries
   no schedule line and the fallback applies.
4. No entry exists for 16 September or later.

### Scenario 13: A flexible-hours schedule produces one row a day

**Given** an employee whose version names a flexible-hours schedule prescribing three hours a day and
twenty-one hours a week, contract 1 September 2025 to 15 September 2025.

**When** generation is requested for 1 September 2025 to 30 September 2025.

**Then** fifteen entries exist, dated 1 to 15 September inclusive, each of 3.000 hours, weekends
included.

### Scenario 14: A schedule eight hours ahead of the universal scale

**Given** an employee on a schedule whose zone is Asia/Hong Kong, with lines Monday to Friday
07:00–11:00 and 13:00–17:00, eight hours a day; a company closure covering the whole of 2 August 2023
local, carrying a kind with the absence flag; contract started 1 August 2023.

**When** generation is requested for 1 August 2023 to 2 August 2023.

**Then**

1. Exactly two entries exist.
2. The first is dated 1 August 2023 with a duration of 8.000 hours, although its earliest interval
   begins at 2023-07-31 23:00 on the universal time scale.
3. The second is dated 2 August 2023 with a duration of 8.000 hours and the closure's kind.
4. No entry is dated 31 July 2023.

### Scenario 15: Three versions in one month

**Given** the setting of scenario 4 after its second version exists.

**When** a third Employee Version is created, dated 20 January 2025, naming the forty-hour schedule
again, and generation is requested for 1 January 2025 to 31 January 2025.

**Then**

1. Twenty-three entries exist for the month.
2. Their durations total 178.000 hours.
3. Seven entries of 8.000 hours belong to the first version, six entries of 7.000 hours to the second,
   and ten entries of 8.000 hours to the third.

### Scenario 16: A version created retroactively re-points a month

**Given** an employee with a single version dated 1 January 2023 and January 2024 already generated:
twenty-three entries, all pointing at that version.

**When** a second Employee Version dated 1 December 2023 is created, and generation is requested for 1
January 2024 to 31 January 2024.

**Then**

1. Twenty-three live entries exist for January 2024.
2. Every one of them points at the **second** version.
3. The entries that pointed at the first version are archived and cancelled.

### Scenario 17: A version with no contract start date generates nothing

**Given** an employee whose only version has a version date of 1 January 2024 and **no** contract start
date.

**When** generation is requested for 1 September 2025 to 30 September 2025.

**Then**

1. No entry is created.
2. No message is shown and no error is raised.
3. The version's last generation date is nevertheless written to today, because it is written before
   the version is examined.

### Scenario 18: A schedule with no lines generates nothing

**Given** an employee whose version names a working schedule carrying no line at all, contract started
1 September 2019.

**When** generation is requested for 1 July 2020 to 30 September 2020.

**Then** no entry is created, and an absence request validated over that period likewise produces no
entry.

### Scenario 19: A schedule line labelled with an absence kind

**Given** a working schedule with two lines on Monday: a morning line 08:00–12:00 carrying `WORK100`,
and an afternoon line 13:00–17:00 carrying a kind that **does** carry the absence flag.

**When** the schedule's working hours are counted for Monday 2 September 2024, and generation is run
for that day.

**Then**

1. The schedule's working-hours count for that day is 4.000 hours, because a line whose kind carries
   the absence flag is excluded from the global attendances.
2. Generation produces two entries for that Monday: one of 4.000 hours of kind `WORK100`, and one of
   4.000 hours of the absence-flagged kind.
3. The schedule's hours-per-week figure excludes the afternoon line.

### Scenario 20: Two kinds on one day produce two rows

**Given** a working schedule whose Monday carries four lines: 08:00–11:00 and 11:00–12:00 both with a
kind `ENTRY_TYPE1`, 13:00–16:00 with `ENTRY_TYPE1`, and 16:00–17:00 with a kind `ENTRY_TYPE2`; neither
kind carries the absence flag.

**When** generation is run for Monday 2 September 2024.

**Then**

1. Exactly two entries exist for that day.
2. One carries `ENTRY_TYPE1` and a duration of 7.000 hours, the sum of 3.000, 1.000 and 3.000.
3. The other carries `ENTRY_TYPE2` and a duration of 1.000 hours.
4. The day totals 8.000 hours.

### Scenario 21: A duration that ends one microsecond short of the hour

**Given** the standard fixture.

**When** a value set running from 2023-10-01 09:00:00.000000 to 2023-10-01 09:59:59.999999 is put
through the post-processing pass and created.

**Then** the resulting entry carries a duration of exactly 1.000 hours, because the span of
3599.999999 seconds is rounded to 3600 whole seconds before being divided by 3600.

---

## 3. Conflicts and validation

### Scenario 22: An entry with no kind cannot be validated

**Given** the standard fixture.

**When** a work entry is created for 1 September 2025 with a duration of 4.000 hours, and its kind is
then cleared.

**Then**

1. The entry is in the state `conflict`.
2. Invoking the validation operation on it returns false and leaves it in `conflict`.
3. When a kind is set on it, the write triggers the check, the reset pass returns it to `draft`, the
   re-check finds nothing, and invoking the validation operation now returns true and leaves it in
   `validated`.

### Scenario 23: A day of exactly twenty-four hours is accepted

**Given** the standard fixture, with no entry on 1 January 2024.

**When** an entry of 8.000 hours and an entry of 16.000 hours are created for the employee on 1
January 2024.

**Then** both are in the state `draft`: the day totals exactly 24.000 hours, and the condition is
strictly greater than twenty-four.

**And when** the second is instead created with 17.000 hours, both are in the state `conflict`.

### Scenario 24: A duration of zero is refused

**Given** the standard fixture.

**When** a work entry is created with a duration of 0.

**Then** the create is refused with the message "Duration must be positive and cannot exceed 24
hours." and no entry exists.

### Scenario 25: A duration above twenty-four hours is refused

**Given** the standard fixture.

**When** a work entry is created with a duration of 24.5.

**Then** the create is refused with the same message, "Duration must be positive and cannot exceed 24
hours."

### Scenario 26: An entry on a day that already holds a validated entry

**Given** an employee with one entry of 8.000 hours on 15 January 2024 in the state `validated`, and
the acting company being the employee's company.

**When** a second entry of 2.000 hours is created for the same employee on 15 January 2024.

**Then**

1. The new entry is in the state `conflict`.
2. The validated entry stays `validated`: entries already validated are excluded from the selection of
   the validation operation, and the day-already-validated condition only marks the entries being
   checked.
3. The day's total is 10.000 hours, which is within the range, so the over-twenty-four condition does
   not fire; the conflict comes from the validated-day condition alone.

### Scenario 27: An absence entry entirely outside the schedule

**Given** the standard fixture, and a work entry kind carrying the absence flag.

**When** two work entries are created for Saturday 13 October 2018, a day the schedule does not cover:
one of kind `WORK100` for 1.000 hour, and one of the absence-flagged kind for 1.000 hour.

**Then**

1. The absence-flagged entry is in the state `conflict`, because its whole-day interval does not
   intersect the schedule at all.
2. The `WORK100` entry is **not** put in conflict by this condition: the condition applies only to
   entries whose kind carries the absence flag.

### Scenario 28: The outside-schedule condition does not apply to a flexible-hours version

**Given** an employee whose version names a flexible-hours schedule.

**When** an entry of an absence-flagged kind is created for a Sunday.

**Then** the entry is **not** put in conflict by the outside-schedule condition, because the version's
schedule is skipped by that test.

### Scenario 29: The outside-schedule condition does not apply to a French part-time employee

**Given** a company whose country is France, an employee whose working schedule differs from the
company's, and an entry of an absence-flagged kind on a day the employee does not work.

**When** the conflict check runs.

**Then** the outside-schedule condition does not mark the entry, because every entry of the selection
is exempt.

### Scenario 30: Validation is all or nothing

**Given** five entries of one employee for one week, four of which are sound and one of which has no
kind.

**When** the validation operation is invoked over all five.

**Then**

1. The operation returns false.
2. **None** of the five is in the state `validated`.
3. The one with no kind is in the state `conflict`; the other four remain `draft`.

### Scenario 31: A conflict clears itself when the cause goes away

**Given** the result of scenario 3, with both entries of 1 September in conflict.

**When** the seventeen-hour manual entry is archived.

**Then**

1. Writing the archived flag also writes the state `cancelled` on it.
2. The archived flag is one of the five fields that trigger the check, so the reset pass runs, the
   remaining eight-hour entry returns to `draft`, and the re-check finds a day total of 8.000 hours.
3. The eight-hour entry ends in the state `draft`.

---

## 4. Absence interactions

### Scenario 32: Validating an absence archives the entry it covers

**Given** an employee whose version has generated-from and generated-to markers surrounding 10 to 12
October 2019, and one work entry of 1.000 hour on 11 October 2019.

**When** an absence request covering 10 October 09:00 to 12 October 18:00 is validated.

**Then**

1. The pre-existing entry of 11 October has its archived flag false.
2. New entries exist carrying the absence kind's work entry kind, which carries the absence flag, and
   linked to the request.
3. None of the new entries is in the state `conflict`.

### Scenario 33: Refusing an approved absence restores the attendance

**Given** an employee whose version markers surround 10 October 2019, an absence request covering
10 October 06:00 to 18:00 that has been validated, and generation run for that day so that an absence
entry linked to the request exists.

**When** the request is refused.

**Then**

1. The absence entry has its archived flag false and its state `cancelled`.
2. Exactly one live entry exists for 10 October 2019, an ordinary attendance entry, newly created.
3. That entry is not in the state `conflict`.
4. The previously archived attendance entry, if there was one, stays archived: the platform generates
   a new row rather than reviving the old one.

### Scenario 34: An absence with a validated entry cannot be cancelled by its requester

**Given** an employee who is also a login user, an absence request they created covering 22 to 25
March 2022 that has been validated, and generation run for 21 to 25 March 2022.

**When** the state of the request's cancellability is read at three moments:

**Then**

1. Before any work entry exists, the request is cancellable.
2. After the work entries exist but before they are validated, the request is still cancellable.
3. After the validation operation has moved them to `validated`, the request is **not** cancellable.

### Scenario 35: Cancelling a work entry refuses its absence request

**Given** a validated absence request and one live work entry linked to it in the state `draft`.

**When** the state `cancelled` is written onto that entry.

**Then**

1. The request is refused first, before the state write happens.
2. The refusal archives every entry linked to the request and regenerates the ordinary attendance of
   those days.
3. The entry ends archived and cancelled.

### Scenario 36: An absence beyond the generated window produces nothing yet

**Given** an employee whose generated-to marker reads 30 September 2025 and an absence request
covering 10 to 12 December 2025.

**When** the request is validated.

**Then**

1. No work entry is created for December 2025: the request's start is later than the generated-to
   marker.
2. When generation later reaches December 2025, the absence rows appear then, labelled with the
   request's work entry kind and linked to it.

### Scenario 37: A company absence generated for everybody

**Given** an employee whose August 2022 has been generated and carries exactly one distinct work entry
kind.

**When** a company-wide absence is generated for 8 August 2022 through the absence domain's own
multiple-generation tool.

**Then** the employee's August 2022 day book carries exactly two distinct work entry kinds: the
ordinary attendance kind and the kind of the company absence.

### Scenario 38: Two absences of different kinds on one day

**Given** an employee on a forty-hour schedule and two absence kinds, one whose work entry kind is
`PAID` and one whose work entry kind is `UNPAID`, both carrying the absence flag.

**When** an absence from 08:00 to 09:00 of the first kind and an absence from 09:00 to 10:00 of the
second kind are both validated for 10 September 2024, and generation is run for that day.

**Then**

1. Three entries exist for the day.
2. One carries `PAID` with a duration of 1.000 hour.
3. One carries `UNPAID` with a duration of 1.000 hour.
4. One carries the ordinary attendance kind with a duration of 6.000 hours.

### Scenario 39: An absence on a flexible schedule spanning four days

**Given** an employee whose version names a flexible schedule of forty hours a week and eight hours a
day, and an absence kind whose work entry kind is `PAID`.

**When** an absence covering 10 to 13 September 2024 is validated and generation is run for 9 to 14
September 2024.

**Then** four entries of kind `PAID` exist, their durations totalling 32.000 hours, that is the number
of days multiplied by the daily hours.

### Scenario 40: Changing the schedule after an absence keeps the absence

**Given** an employee on a twenty-hour schedule, Monday to Friday 08:00–12:00, with October 2019
generated; and a validated one-day absence on 10 October 2019 that produced one entry of 4.000 hours
linked to the request.

**When** the version's working schedule is changed to the forty-hour schedule.

**Then**

1. Exactly one entry still exists for 10 October 2019.
2. It still carries the absence kind's work entry kind and still links to the request.
3. Its duration is now 8.000 hours, because the absence is measured against the new schedule.

---

## 5. Regeneration

### Scenario 41: A forced regeneration discards manual changes

**Given** the result of scenario 1, and the entry of 2 September 2025 edited by hand to 3.000 hours.

**When** the regeneration wizard is run for that employee over 1 September 2025 to 7 September 2025.

**Then**

1. The five pre-existing entries are archived and cancelled.
2. Five new entries exist, dated 1 to 5 September, each of 8.000 hours of kind `WORK100` in the state
   `draft`.
3. The hand-edited value is gone; the archived row still records it.

### Scenario 42: The wizard refuses incomplete criteria

**Given** a regeneration wizard carrying a from-date and a to-date but no employee.

**When** the regeneration operation is invoked without the skip marker.

**Then** it is refused with:

> "In order to regenerate the work entries, you need to provide the wizard with an employee_id, a date_from and a date_to."

### Scenario 43: The wizard refuses a range outside the generated window

**Given** a regeneration wizard whose employees have an earliest available date of 1 September 2025
and a latest available date of 30 September 2025, with a from-date of 1 August 2025.

**When** the regeneration operation is invoked without the skip marker.

**Then** it is refused with the message beginning "The from date must be >= " and naming both
boundary dates in the acting user's date format.

### Scenario 44: The wizard refuses when nothing can be regenerated

**Given** a regeneration wizard whose single selected employee holds a validated entry inside the
requested range.

**When** the regeneration operation is invoked without the skip marker.

**Then** it is refused with:

> "No work entry can be regenerated in this range of dates and these employees."

### Scenario 45: A schedule change on an employee with validated entries regenerates nothing

**Given** an employee holding one validated entry on 15 January 2024, and a regeneration wizard for
that employee over 1 January 2024 to 31 January 2024.

**When** the regeneration operation is invoked **with** the skip marker, as the automatic regeneration
after a version change does.

**Then**

1. The three guards do not fire.
2. The employee is removed from the selection because they hold a validated entry.
3. The selection is empty, so the generation entry point is **not** called at all.
4. No entry is created, changed, archived or deleted.

### Scenario 46: The calendar reset collapses days into runs

**Given** a selection of employee A on 3, 4, 5 and 9 March and employee B on 3, 4 and 5 March.

**When** the regeneration operation is invoked with that list of day slots.

**Then**

1. Exactly two forced generations are issued.
2. The first covers 3 March to 5 March for employees A and B together.
3. The second covers 9 March to 9 March for employee A alone.
4. None of the three guards is evaluated.

### Scenario 47: A forced regeneration of one month leaves the neighbouring month alone

**Given** an employee in Europe/Brussels with January 2024 generated by a forced run.

**When** February 2024 is generated by a second forced run, for 1 February to 29 February 2024.

**Then** the entries of January 2024 are exactly the ones the first run produced: none of them is
archived, cancelled or replaced, despite the window of the February run beginning on 31 January on the
universal time scale.

---

## 6. The catalogue of kinds

### Scenario 48: A duplicate payroll code is refused

**Given** a universal work entry kind with the payroll code `WORKTEST200`.

**When** a second kind is created with the same payroll code and no country.

**Then** the create is refused with a message beginning "The same code cannot be associated to
multiple work entry types (" and naming `WORKTEST200`.

### Scenario 49: The same payroll code in two different countries is allowed

**Given** a kind with the payroll code `SICK25` whose country is Slovakia.

**When** a kind with the payroll code `SICK25` whose country is Poland is created.

**Then** the create succeeds: the uniqueness search considers only the countries present in the write
and the empty country.

### Scenario 50: A universal code and a country code may not collide

**Given** a kind with the payroll code `LEAVE120` and no country.

**When** a kind with the payroll code `LEAVE120` whose country is Belgium is created.

**Then** the create is refused, because the search deliberately includes kinds with no country.

### Scenario 51: The country of the ordinary-attendance kind may not change

**Given** the shipped ordinary-attendance kind, payroll code `WORK100`.

**When** a country is written onto it.

**Then** the write is refused with:

> "You can't change the country of this specific work entry type."

### Scenario 52: The country of a kind in use may not change

**Given** a kind with the payroll code `WORKTEST200` and at least one work entry referencing it.

**When** a country is written onto it, outside the installation of shipped data.

**Then** the write is refused with:

> "You can't change the Country of this work entry type cause it's currently used by the system. You need to delete related working entries first."

### Scenario 53: Archiving a kind leaves its entries intact

**Given** a kind with ten work entries referencing it.

**When** the kind is archived.

**Then**

1. The ten entries still exist, still reference the kind and keep their durations, states and pay
   rates.
2. The kind no longer appears in any selection list or in the default search.
3. No entry changes state.

### Scenario 54: The working-time indicator is the negation of the absence flag

**Given** a kind whose absence flag is false.

**When** the working-time indicator is read, and then written as false.

**Then** the indicator reads true initially, and after the write the absence flag reads true and the
indicator reads false.

---

## 7. Splitting, filters, companies and permissions

### Scenario 55: Splitting an eight-hour entry into five and three

**Given** one entry of 8.000 hours on 3 March 2025 of kind `WORK100`.

**When** the split operation is invoked with a duration of 3.000 hours, the kind `OVERTIME` and a
description.

**Then**

1. The original entry now carries 5.000 hours and still carries `WORK100`.
2. A second entry exists carrying 3.000 hours, the kind `OVERTIME`, the same employee, version, date
   and company, and the state `draft`.
3. The day totals 8.000 hours and is not in conflict.

### Scenario 56: Splitting an entry shorter than an hour is refused

**Given** one entry of 0.5 hours.

**When** the split operation is invoked.

**Then** it is refused with: "You can't split a work entry with less than 1 hour."

### Scenario 57: Splitting off more than the whole is refused

**Given** one entry of 4.000 hours.

**When** the split operation is invoked with a duration of 4.000 hours.

**Then** it is refused with: "Split work entry duration has to be less than the existing work entry
duration."

### Scenario 58: A work entry takes the company of its employee, not of the acting user

**Given** two companies, A and B, both accessible to the acting user, whose acting company is A; and an
employee belonging to company B.

**When** a work entry is created for that employee with no company supplied.

**Then** the entry's company is B.

### Scenario 59: A user cannot see another company's work entries

**Given** the records of scenario 58 and an acting user whose acting companies are A alone.

**When** the work entries are listed.

**Then** the entry of company B is not returned, by the multi-company record rule.

### Scenario 60: An officer cannot delete a work entry

**Given** a work entry in the state `draft` and an acting user who is a Human Resources Officer and not
a member of the Settings Administrator group.

**When** the user attempts to delete the entry.

**Then** the deletion is refused by the access matrix, before rule `WKE-035` is reached. The officer
archives it instead, which cancels it.

### Scenario 61: A validated entry cannot be deleted even by the Settings Administrator group

**Given** a work entry in the state `validated` and an acting user in the Settings Administrator group.

**When** the user attempts to delete the entry.

**Then** the deletion is refused with: "This work entry is validated. You can't delete it."

### Scenario 62: A user may pin an employee only once

**Given** a calendar filter row for user U and employee E.

**When** a second row for the same pair is created.

**Then** the create is refused with: "You cannot have the same employee twice."

**And when** the first row is archived and a second row for the same pair is created, the create is
still refused, because the constraint does not exclude archived rows.

### Scenario 63: A user may not change another user's calendar filter

**Given** a calendar filter row belonging to user U.

**When** user V, an internal user, attempts to write to it.

**Then** the write is refused by the record rule. Reading it is **not** refused, because read is
deliberately left out of the rule.

### Scenario 64: Generation refuses to run without a time zone

**Given** an employee whose version's schedule, whose own schedule and whose company's schedule all
name no time zone.

**When** generation is requested.

**Then** the run fails with: "Missing timezone for work entries generation."

### Scenario 65: Removing a version deletes its non-validated entries

**Given** a version with ten entries inside its effective window, all in the state `draft`.

**When** the version is removed.

**Then** the ten entries are deleted, not archived.

**And given** instead that one of the ten is in the state `validated`, the removal fails, because the
entry's link to the version refuses deletion while entries remain and because the validated entry is
excluded from the cancellation.

### Scenario 66: The state is not carried over by a copy

**Given** a work entry in the state `conflict`.

**When** it is duplicated.

**Then** the copy is in the state `draft`, and it carries the same description, employee, version,
date, duration, kind, company, pay rate and archived flag as the original.

### Scenario 67: Writing the validated state together with the archived flag

**Given** a work entry in the state `draft`.

**When** the state `validated` and the archived flag false are written in the same operation.

**Then** the entry ends in the state `cancelled` with its archived flag false, because the flag rule
runs after the state rule. This is the **compatibility finding** recorded in
[state-machines.md, section 1.2](state-machines.md#12-the-coupling-with-the-archived-flag).

### Scenario 68: The daily job fills the current and the next month

**Given** today is 12 September 2025, and one version whose markers are both 12 September 2025 at
midnight.

**When** the daily job runs.

**Then**

1. Its period is 1 September 2025 to 31 October 2025.
2. The version qualifies, because its generated-from marker is later than the period start.
3. Generation runs, not forced, and the day book of September and October is written.
4. The version's last generation date reads 12 September 2025, so a second run of the job on the same
   day does nothing.

### Scenario 69: The daily job processes one company at a time

**Given** one hundred and fifty versions with something to do, belonging to two companies.

**When** the daily job runs.

**Then**

1. Only the versions of the first company are considered.
2. At most one hundred of them are generated in this run.
3. The job re-triggers itself, because the count of versions with something to do exceeded one
   hundred.

### Scenario 70: The French gap filling adds the days the employee does not work

**Given** a company in France whose working schedule is Monday to Friday 08:00–12:00 and 13:00–17:00;
an employee working Mondays and Wednesdays on the same hours; and a validated absence requested from
6 September 2021 to 8 September 2021 whose end was adjusted to 10 September 2021 by the French
part-time rule.

**When** the value sets for 6 September 2021 to 10 September 2021 are computed.

**Then**

1. Ten value sets are produced: four from the ordinary path and six from the gap filling.
2. Each of the six gap value sets carries the absence kind's work entry kind and a link to the request.

**And when** the value sets are computed for 6 September 2021 to 9 September 2021 instead, eight are
produced: four from the ordinary path and four from the gap filling.
