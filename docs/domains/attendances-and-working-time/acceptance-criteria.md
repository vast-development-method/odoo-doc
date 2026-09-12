# Attendances and Working Time — Acceptance criteria

Numbered, independently verifiable scenarios a rebuild must pass. Each is written as Given,
When and Then with concrete records, concrete inputs and exact resulting values. Identifiers
have the form `AWT-AC-nnn` and are stable: a scenario is never renumbered.

## The fixture

Unless a scenario says otherwise:

- **Company A** owns the default Working Schedule `Standard 40 hours/week`, whose time zone is
  **universal time** and whose pattern is Monday to Friday, morning `08:00`–`12:00`, break
  `12:00`–`13:00`, afternoon `13:00`–`17:00`. Its extra-hours validation is "Automatically
  Approved"; absence management is off; automatic check-out is off; device and location
  tracking is off; the full-time reference is forty hours.
- **Employee E** belongs to company A, owns a Resource whose own time zone is universal time,
  follows the company default schedule, and has one Employee Version effective from 1 January
  2020 with a contract starting the same day and naming rule set **R**.
- **Rule set R** holds one quantity rule: period day, expected quantity read from the employee's
  schedule, no tolerances, not paid, rate one.
- Instants written without a zone are universal time. Because the fixture schedule's zone is
  universal time, the wall-clock times of the pattern and the stored instants coincide; every
  scenario that needs them to differ names its zone explicitly.
- "The balance" means the employee's total approved extra hours, that is the sum of the encoded
  amounts of the approved lines.

---

## Working schedules

**AWT-AC-001: Weekly and daily averages of a plain schedule.** Given a schedule with three
full-day periods, Monday, Tuesday and Wednesday, each `08:00`–`16:00`. When it is saved. Then
the total hours per week is `24.00` and the average hours per day is `8.00`.

**AWT-AC-002: Averages follow an edit.** Given the schedule of AWT-AC-001. When the Monday
period is changed to start at `11:00`. Then the total hours per week becomes `21.00` and the
average hours per day becomes `7.00`, both before the form is saved and after.

**AWT-AC-003: Averages on a duration-based schedule.** Given a duration-based schedule with
three full-day periods on Monday, Tuesday and Wednesday, each of length four hours. When it is
saved. Then the total hours per week is `12.00` and the average hours per day is `4.00`.

**AWT-AC-004: A length edit rewrites the clock times.** Given the duration-based counterpart of
the schedule of AWT-AC-001. When the length of the Monday full-day period is set to `5`. Then
the total hours per week becomes `21.00`, the average hours per day becomes `7.00`, and the
Monday period runs from `09:30` to `14:30`.

**AWT-AC-005: A two-week schedule averaging thirty-eight hours.** Given a two-week schedule
whose first week is Monday to Friday `08:00`–`12:00` and `13:00`–`17:00` with a break, and
whose second week is the same except that Friday holds only the morning. When it is saved. Then
the total hours per week is `38.00`, the working days per week is `5`, the average hours per
day is `7.60`, the work-time rate against a forty-hour reference is `95.00`, and the schedule
is not full time.

**AWT-AC-006: Switching to two weeks duplicates the pattern.** Given a fixed schedule with
sixteen working periods and a company whose default schedule is in two-week mode. When a new
schedule is created from the form. Then it is in two-week mode and holds sixteen periods in the
first week and sixteen in the second, plus the two section markers.

**AWT-AC-007: Switching the encoding restores the company hours, not the built-in ones.** Given
a company whose default schedule holds only morning periods running `09:00`–`18:00`, and a
schedule created from that default and switched to two weeks. When the schedule is switched to
duration-based entry and then back to start-and-end entry. Then every period runs from `09:00`
to `18:00`, and not from the built-in `08:00`–`12:00` pattern.

**AWT-AC-008: Overlapping periods are refused.** Given a schedule with a Monday period from
`08:00` to `12:00`. When a second Monday period from `11:00` to `13:00` is added and saved.
Then the save is refused with "Attendances can't overlap." and neither period is stored.

**AWT-AC-009: Touching periods are accepted.** Given the same schedule. When a second Monday
period from `12:00` to `17:00` is added and saved. Then the save succeeds, because the overlap
test adds one microsecond to each start.

**AWT-AC-010: Periods of different weeks never conflict.** Given a two-week schedule. When a
Monday period from `08:00` to `12:00` exists in the first week and another Monday period from
`08:00` to `12:00` exists in the second. Then the save succeeds.

**AWT-AC-011: Sections must come first in two-week mode.** Given a two-week schedule holding
section markers. When a working period is given a sequence lower than every section marker and
the schedule is saved. Then the save is refused with "In a calendar with 2 weeks mode, all
periods need to be in the sections."

**AWT-AC-012: A section marker may not be deleted.** Given a two-week schedule open in a form.
When one of the two section markers is removed from the period list. Then the change is refused
with "You can't delete section between weeks." and the list is left unchanged.

**AWT-AC-013: A break is forbidden on a duration-based schedule.** Given a duration-based
schedule. When a period named `Monday Lunch` with the period kind "Break" is added. Then the
save is refused with "Monday Lunch is a break attendance, You should not have such record on
duration based calendar".

**AWT-AC-014: Hour clamping.** Given a working period open in a form. When the start hour is
typed as `-2` and the end hour as `30`. Then the start hour becomes `0` and the end hour
becomes `24`. When the start hour is then typed as `18` while the end hour reads `16`. Then the
end hour is raised to `18`.

**AWT-AC-015: Half-day classification.** Given a schedule whose average hours per day is `8`,
so that the threshold is `6`. When a morning period of four hours is saved. Then its length in
days is `0.5`. When a morning period of seven hours is saved. Then its length in days is `1`.
When a full-day period of two hours is saved. Then its length in days is `1`. When a break of
one hour is saved. Then its length in hours is `0` and its length in days is `0`.

**AWT-AC-016: The week in force.** Given a two-week schedule. When the week type of Monday
10 November 2025 is computed. Then it is the **first** week. When the week type of Monday
3 November 2025 is computed. Then it is the **second** week. When the week type of Monday
17 November 2025 is computed. Then it is the **second** week, and Monday 24 November 2025 is
the first week again: the pattern alternates every seven days from the first day of the
calendar and never from a stored counter.

**AWT-AC-017: The work-time rate without a reference.** Given a schedule whose full-time
reference is `0` and whose total hours per week is `20`. When the rate is read. Then it is
exactly `100.00`.

**AWT-AC-018: A company always has a default schedule.** Given no company. When a company is
created without naming a working schedule. Then a schedule named `Standard 40 hours/week` owned
by that company exists, holds the fifteen built-in periods, and is the company's default.

---

## Intervals, counting and planning

**AWT-AC-019: Working intervals of a plain week.** Given the fixture schedule and no exclusion.
When the working intervals are requested from Monday 10 November 2025 `00:00` to Friday
14 November 2025 `23:59:59`. Then ten intervals are returned, two per day, and their total is
`40` hours.

**AWT-AC-020: Break intervals.** Given the same query with break periods requested instead.
Then five intervals of one hour each are returned, totalling `5` hours.

**AWT-AC-021: A closure and a half-day absence.** Given a closure covering the whole of Tuesday
11 November 2025 for company A, and a personal exclusion for employee E from Thursday
13 November `08:00` to `12:00`. When the working intervals of E are requested over that week.
Then their total is `28` hours and the duration measurement returns `3.5` days.

**AWT-AC-022: A partial absence produces a partial day.** Given the fixture and a personal
exclusion for E on Friday 14 November from `08:00` to `10:00`. When the duration measurement is
requested for that Friday. Then it returns `6` hours and `0.75` day: the morning contributes
0.5 × ( 2 ÷ 4 ) = 0.25 and the afternoon contributes 0.5 × ( 4 ÷ 4 ) = 0.5.

**AWT-AC-023: A closure of another company does not apply.** Given a closure of company B
covering 29 August 2025 and a fully flexible resource of company A. When that resource's
attendance intervals are requested from 28 to 31 August 2025. Then all four days are returned
whole: the closure of company B removes nothing.

**AWT-AC-024: Hour counting with and without exclusions.** Given the fixture of AWT-AC-021.
When the hour count of the schedule alone is requested over that week with exclusions
subtracted. Then it returns `28`. When it is requested without subtracting them. Then it
returns `40`.

**AWT-AC-025: Unusual days.** Given a forty-hour schedule and a closure on Wednesday 29 May
2019. When the unusual days from Monday 27 to Friday 31 May 2019 are requested for that
company. Then the answer is 27 May false, 28 May false, 29 May **true**, 30 May false, 31 May
false.

**AWT-AC-026: Forward planning by hours.** Given the fixture schedule and no exclusion. When
twenty working hours are planned forward from Monday 10 November 2025 `08:00`. Then the answer
is Wednesday 12 November 2025 `12:00`.

**AWT-AC-027: Backward planning by hours.** Given the same. When six working hours are planned
backward from Friday 14 November 2025 `17:00`. Then the answer is Friday 14 November 2025
`10:00`.

**AWT-AC-028: Backward planning by days.** Given the same. When three working days are planned
backward from Monday 10 November 2025 `08:00`. Then the answer is Wednesday 5 November 2025
`13:00`, the start of the last working period of that day.

**AWT-AC-029: Forward planning by days.** Given the same. When three working days are planned
forward from Monday 10 November 2025 `08:00`. Then the answer is Wednesday 12 November 2025
`12:00`, the end of the interval that completed the count.

**AWT-AC-030: Planning by days honours exclusions.** Given the fixture plus a closure on
Tuesday 11 November 2025. When three working days are planned forward from Monday 10 November
2025 `08:00` with exclusions subtracted. Then the answer is Thursday 13 November 2025 `12:00`.

**AWT-AC-031: Planning returns no result beyond the horizon.** Given a schedule with no working
period at all. When any positive quantity of hours is planned forward. Then no result is
returned, after at most 1400 days of search, rather than an endless search.

**AWT-AC-032: Snapping a span to the schedule.** Given a schedule with periods `08:00`–`13:00`
and `14:00`–`17:00` and a resource following it. When the pair `09:00` and `18:00` of the same
day is snapped to the schedule. Then the result is `08:00` and `17:00`.

**AWT-AC-033: The closest working moment rejects a bound with no zone.** Given any schedule.
When the closest working moment is requested for an instant that carries no zone. Then the call
fails with "Provided datetimes needs to be timezoned".

**AWT-AC-034: Unavailable intervals of a fixed resource.** Given the fixture resource and
Monday 10 November 2025 from `00:00` to `23:59:59`. When the unavailable intervals are
requested. Then they are `00:00`–`08:00`, `12:00`–`13:00` and `17:00`–`23:59:59`.

**AWT-AC-035: Unavailable intervals of a flexible resource.** Given a flexible resource with
one personal exclusion from `09:00` to `11:00` on that day. When the unavailable intervals are
requested. Then only that exclusion is returned, widened to the whole day because the resource
is flexible.

---

## Flexible schedules

**AWT-AC-036: Flexible intervals fill the weekly budget.** Given a flexible schedule of thirty
hours per week and seven hours per day in universal time. When the attendance intervals are
requested from Monday 2 June 2025 `00:00` to Saturday 7 June 2025 `23:59:59`. Then five
intervals of `7`, `7`, `7`, `7` and `2` hours are returned, the first four centred on `12:00`.

**AWT-AC-037: Flexible intervals stay inside the query.** Given the same schedule. When the
intervals are requested from Monday 2 June `11:00` to Saturday 7 June `13:00`. Then no interval
starts before `11:00` on the first day and none ends after `13:00` on the last.

**AWT-AC-038: A fully flexible resource is always available.** Given a resource with no
schedule at all whose own zone is `America/New_York`. When the attendance intervals are
requested from `18:00` to `21:00`. Then a single interval from `18:00` to `21:00` is returned,
whose synthetic period reports three hours and `0.125` day, that is 3 ÷ 24.

**AWT-AC-039: Flexible weekly hours.** Given a flexible schedule of thirty-eight hours per week
and `7.6` hours per day against a forty-hour reference, and a resource on it. When the flexible
working hours are measured from Monday 28 July to Sunday 3 August 2025. Then the result is
`38.0` and the weekly cap of that week is `38.0`.

**AWT-AC-040: Flexible caps after absences.** Given a flexible schedule of forty hours per week
and eight hours per day, a resource on it, an absence on Tuesday 29 July 2025 and another from
Thursday 31 July to Friday 1 August 2025. When the flexible intervals and caps are measured
from Monday 28 July to Sunday 3 August 2025. Then the working intervals are the whole days of
28 July, 30 July, 2 August and 3 August; the daily caps are `8`, `0`, `8`, `0`, `0`, `8`, `8`;
and the weekly cap is `16`.

**AWT-AC-041: Weekly caps across a year boundary.** Given the same schedule. When the caps are
measured from Friday 26 December 2025 to Thursday 1 January 2026. Then two weekly caps are
returned, one for the week that contains 26 December 2025 and one for the week that contains
1 January 2026, each of `40.0`.

**AWT-AC-042: The first day of the week follows the reader's language.** Given a flexible
schedule of forty hours per week and eight hours per day, and a reader whose active language
starts the week on Sunday. When the flexible working hours are measured from Sunday 12 April
2026 to Saturday 18 April 2026. Then the result is `40.0`, because the whole span is one week
for that reader.

---

## Attendance integrity

**AWT-AC-043: A check-out before the check-in is refused.** Given employee E. When an
attendance is created with a check-in at `12:00` and a check-out at `11:00` on the same day.
Then the creation is refused with "\"Check Out\" time cannot be earlier than \"Check In\"
time."

**AWT-AC-044: A second open record is refused.** Given an open attendance of E checked in at
`10:00`. When a second attendance of E is created at `11:00` with no check-out. Then the
creation is refused with the message that begins "Cannot create new attendance record for" and
names E and the instant `10:00`.

**AWT-AC-045: An overlapping record is refused.** Given a closed attendance of E from `07:30`
to `09:00`. When an attendance of E from `08:30` to `09:30` is created. Then the creation is
refused and nothing is stored.

**AWT-AC-046: An edit that creates an overlap is refused.** Given an open attendance of E
checked in at `10:00` and a closed attendance from `11:00` to `12:00`. When the open one is
given a check-out of `11:30`. Then the write is refused and the record stays open.

**AWT-AC-047: A zero-length record is accepted.** Given employee E with no open record. When an
attendance is created whose check-in and check-out are the same instant. Then the record is
created, its worked hours are `0.00` and it carries no extra-hours line.

**AWT-AC-048: A future record does not change the state.** Given that it is 5 February 2024 at
`11:00`. When an attendance of employee F is created from 10 February `11:00` to `12:00`, and
then an open attendance is created on 5 February at `10:00`. Then F reads `checked_in`. When
the open record is given a check-out at `11:30`. Then F reads `checked_out`, even though the
future record exists.

**AWT-AC-049: Duplication is refused.** Given any attendance. When it is duplicated. Then the
operation fails with "You cannot duplicate an attendance." and no record is created.

**AWT-AC-050: A check-out with no open record fails.** Given an employee whose attendance state
says checked in but whose open record has been deleted. When a check-out is requested from a
device. Then the operation fails with "Cannot perform check out on *employee name*, could not
find corresponding check in. Your attendances have probably been modified manually by human
resources."

**AWT-AC-051: The display name of an attendance.** Given a reader whose zone is universal time
and whose time format shows hours, minutes and seconds, and an attendance from `08:00` to
`09:00`. Then its display name is "01:00 (08:00:00-09:00:00)". Given a reader whose format is a
twelve-hour clock. Then it is "01:00 (08:00:00 AM-09:00:00 AM)". Given an open attendance
checked in at `09:00`. Then it is "From 09:00:00".

**AWT-AC-052: Archiving closes the open record.** Given an open attendance of E checked in on
1 January 2024 at `08:00`. When E is archived at `17:00` that day. Then the record has a
check-out of `17:00` and worked hours of `8.0`, the break having been removed.

**AWT-AC-053: Archiving works without attendance rights.** Given the same, and a user holding
only the human-resources user group. When that user archives E. Then E is archived and the open
record is closed at the instant of archiving.

**AWT-AC-054: The check-in cannot be emptied.** Given an attendance form with an employee
chosen. When the check-in is cleared and the form is saved. Then the save fails because the
check-in is required.

---

## Worked hours

**AWT-AC-055: The break is removed.** Given the fixture schedule. When E checks in at `08:00`
and out at `17:00`. Then the worked hours are `8.00`.

**AWT-AC-056: A partial overlap of the break is removed proportionally.** Given the same. When
E works from `08:00` to `12:30`. Then the worked hours are `4.00`: four and a half elapsed
hours minus the half hour of break inside the span.

**AWT-AC-057: A flexible resource keeps the whole elapsed time.** Given a flexible schedule of
eight hours per day. When the employee works from `10:00` to `22:00`. Then the worked hours are
`12.00`; no break is ever removed for a flexible or fully flexible resource.

**AWT-AC-058: Hours today count from the local midnight.** Given an employee whose own zone is
`Europe/Brussels`, one record from 1 March `22:00` local to 2 March `02:00` local and an open
record from 2 March `11:00` local, evaluated at 2 March `14:00` local. Then the hours today are
`5`, the hours previously today are `2` and the hours of the current stretch are `3`.

**AWT-AC-059: Hours of the current month.** Given that today is 15 October and an employee
whose closed records between 1 October and now total thirty-seven hours, of which four hours
are validated extra hours. Then the hours of the month read `37.0` and the extra hours of the
month read `4.0`; a record that began on 30 September is excluded entirely, and an open record
is excluded entirely.

---

## Extra hours, quantity rules

**AWT-AC-060: No extra hours before the check-out.** Given E and rule set R. When an attendance
is created at `08:00` with no check-out. Then no extra-hours line exists for that day and the
record's extra hours read `0`.

**AWT-AC-061: A simple daily excess.** Given E and R. When E works from `08:00` to `12:00` and
then from `13:00` to `18:00` on a Monday. Then the countable total is nine hours against eight
expected, one line of `1` hour exists for that day against the second record, covering
`17:00`–`18:00`, and the balance is `1`.

**AWT-AC-062: The excess is taken from the end.** Given E and R and a single attendance from
`08:00` to `20:00`. Then the countable total is eleven hours, one line of `3` hours exists, and
it covers the last three hours of the day.

**AWT-AC-063: Working through the break earns nothing.** Given E and R. When E works from
`08:00` to `17:00`, or from `07:00` to `16:00`, or from `09:00` to `18:00`, each time straight
through the scheduled break. Then in each case the countable total is exactly eight hours and
the balance is `0`.

**AWT-AC-064: Several records in one day are summed.** Given E and a rule set with a daily rule
expecting eight hours. When E works from `08:00` to `14:00` and from `14:00` to `20:00`. Then
the day yields four hours of extra time, attributed to the second record.

**AWT-AC-065: A shortfall requires absence management.** Given E, R and absence management
**off**. When E works from `07:00` to `08:00`. Then no line exists, the record's extra hours
read `0` and its regular hours read `1`. Given the same with absence management **on** and a
check-out at `09:00`. Then one line of `−6` exists, the extra hours read `−6` and the regular
hours read `8`.

**AWT-AC-066: Only the least severe shortfall is kept.** Given absence management on and a rule
set with two daily rules expecting eight and ten hours. When E works from `13:00` to `18:00`,
that is five countable hours. Then exactly one line exists and its amount is `−3`.

**AWT-AC-067: Shortfalls of different periods do not add up.** Given absence management on and
a rule set with a daily rule expecting eight hours and a weekly rule expecting forty. When E
works five countable hours on one day of the week and nothing else. Then exactly one line
exists and its amount is `−3`.

**AWT-AC-068: A day repaired by later records loses its shortfall.** Given absence management
on, R, and an attendance of E from `08:00` to `12:00` on 3 January 2023 which produced a line
of `−4`. When a second attendance from `13:00` to `17:00` is added the same day. Then both
records report `0` extra hours and no line remains. When a third record from `18:00` to `19:00`
is added. Then exactly one line of `+1` exists, on the third record. When the third record is
extended to `20:00`. Then it reports `+2`. When the second record is then deleted. Then the
third record reports `−2`.

**AWT-AC-069: The employer tolerance is a threshold.** Given R with an employer tolerance of
`10 ÷ 60`. When E works `07:55`–`12:00` and `13:00`–`17:05`, that is 8.1667 countable hours.
Then no line exists, because the excess of 0.1667 does not **exceed** the tolerance. When the
tolerance is lowered to `4 ÷ 60` and the rule set is regenerated. Then one line of `0.1667`
hours exists, that is the whole ten minutes and not the part above the tolerance.

**AWT-AC-070: The employee tolerance is a threshold.** Given R with an employee tolerance of
`10 ÷ 60` and absence management on. When E works `08:05`–`12:00` and `13:00`–`16:55`, that is
7.8333 countable hours. Then no line exists. When the tolerance is lowered to `4 ÷ 60` and the
rule set is regenerated. Then one line of `−0.1667` hours exists.

**AWT-AC-071: The employer tolerance across several records of a day.** Given R with an
employer tolerance of `0.25`. When E works `07:00`–`08:00` and `12:00`–`20:30` on the first
day, `07:00`–`08:00` and `12:00`–`20:14` on the second, and `07:44`–`12:00` and
`13:30`–`17:44` on the third. Then the extra hours per record, in order, are `0`, `0.5`, `0`,
`0`, `0`, `0.5`.

**AWT-AC-072: The employee tolerance across several records of a day.** Given R with an
employee tolerance of `0.25` and absence management on. When E works `07:00`–`08:00` and
`12:00`–`19:30` on the first day, `07:00`–`08:00` and `12:00`–`19:54` on the second, and
`07:44`–`12:00` and `13:30`–`16:44` on the third. Then the extra hours per record, in order,
are `0`, `−0.5`, `0`, `0`, `0`, `−0.5`.

**AWT-AC-073: A daily rule and a weekly rule combine into three lines.** Given a rule set with
a daily rule expecting nine hours and a weekly rule expecting forty, both with fixed
quantities. When E works Monday and Friday from `08:00` to `19:00` and Tuesday to Thursday from
`08:00` to `17:00`. Then three lines exist: Monday `1`, Friday `3` and Friday `1`; the Monday
line names the daily rule alone, the three-hour Friday line names the weekly rule alone and the
one-hour Friday line names both; the total is `5`.

**AWT-AC-074: The weekly excess reaches back over the whole week.** Given a schedule Monday to
Friday `08:00`–`16:00` with no break, and a rule set with a daily rule and a weekly rule both
reading the employee's schedule, so that the expectations are eight and forty. When E works
`08:00`–`18:00` every working day. Then the balance is `18`: two hours of daily excess on each
of the five days, plus a weekly excess of ten attributed to the last ten countable hours of the
week, of which two coincide with Friday's daily excess.

**AWT-AC-075: Two daily rules with the same quantity produce one line.** Given a rule set with
two daily rules each expecting eight hours. When E works ten countable hours. Then exactly one
line of `2` exists, naming both rules, and not two lines.

**AWT-AC-076: Two daily rules with different quantities produce two lines.** Given a rule set
with daily rules expecting eight and ten hours. When E works twelve countable hours. Then two
lines of `2` each exist and the balance is `4`.

**AWT-AC-077: A partial week produces only the daily excess.** Given the rule set of
AWT-AC-076 plus a weekly rule expecting forty hours. When E works twelve countable hours on
Monday, Tuesday and Wednesday only. Then the balance is `12` and no weekly line exists, because
the week totals thirty-six.

**AWT-AC-078: A flexible employee, daily rule.** Given a flexible schedule of forty hours per
week and eight hours per day and rule set R. When the employee works `08:00`–`16:00`. Then the
extra hours are `0`. When the record is moved to `12:00`–`18:00` with absence management
**off**. Then the extra hours are `0`. When it is moved to `10:00`–`22:00`. Then the extra
hours are `4`.

**AWT-AC-079: A flexible employee, shortfall.** Given the same with absence management **on**.
When the employee works `12:00`–`18:00`. Then the extra hours are `−2`.

**AWT-AC-080: A fully flexible employee never earns extra hours.** Given an employee with no
schedule at all. When the employee works `08:00`–`16:00`, and then from `16:00` on one day to
`09:00` the next. Then the extra hours are `0` in both cases, and no line exists.

**AWT-AC-081: A flexible weekly budget spread over non-consecutive days.** Given a flexible
schedule of sixteen hours per week and eight hours per day, and a daily quantity rule reading
the employee's schedule. When the employee works eight hours on Monday 6 January 2025 and eight
hours on Saturday 11 January 2025. Then both records report `0` extra hours.

**AWT-AC-082: A flexible weekly budget exceeded.** Given a flexible schedule of forty hours per
week and eight hours per day and a weekly rule reading the employee's schedule. When the
employee works 8, 10, 5, 15 and 12 hours on the five days of one week. Then the sum of the
extra hours of that week is `10`.

---

## Extra hours, timing rules

**AWT-AC-083: A non-working day is entirely extra.** Given a rule set with the daily quantity
rule and a timing rule of kind "on any non-working day" with the band `0` to `24`. When E works
from `08:00` to `11:00` on Saturday 2 January 2021. Then one line of `3` exists and the balance
is `3`.

**AWT-AC-084: A non-working day and a working day accumulate.** Given the same rule set. When E
works `08:00`–`19:00` on Saturday 2 January and `07:00`–`17:00` on Monday 4 January 2021. Then
the balance is `12`: eleven hours from the Saturday and one from the Monday. When the Saturday
record is deleted. Then the balance is `1`.

**AWT-AC-085: Outside a named schedule.** Given a rule set with a timing rule of kind "outside
of a specific schedule" naming the company schedule, and a timing rule on working days with the
band `14:00`–`15:00`. When E works from `07:00` to `16:00`. Then two lines of one hour each
exist for that day: `07:00`–`08:00` from the first rule, because the named schedule's work and
break periods together cover `08:00`–`17:00`, and `14:00`–`15:00` from the second.

**AWT-AC-086: Adjacent timing bands stay separate.** Given a rule set with a timing rule on
working days from `17:00` to `21:00` and another from `21:00` to `24:00`. When E works from
`17:00` to `23:59:59`. Then two lines exist, of `4` and `2.9997` hours, each naming its own
rule; the second is displayed as three hours.

**AWT-AC-087: A timing band that wraps midnight.** Given a rule set whose single rule is a
timing rule on working days from `14:00` to `05:00`. When E works from `08:00` to `18:00`. Then
the worked hours are `9.0`, the extra hours are `4.0` and the regular hours are `5.0`.

**AWT-AC-088: The employer tolerance of a timing rule.** Given a rule set with a timing rule on
non-working days with an employer tolerance of one hour. When E works ten minutes on a
Saturday. Then no line exists. When E works exactly one hour on another Saturday. Then no line
exists, because the total does not **exceed** the tolerance. When E works four hours on a third
Saturday. Then one line of `4` exists, not of three.

**AWT-AC-089: A closure belongs to its company.** Given company A with a closure on 11 November
2025 and company B without one, both using a rule set whose single rule treats any non-working
day as extra. When an employee of each works from `08:00` to `17:00` that day. Then the
employee of company A has `9` extra hours and the employee of company B has `0`, that Tuesday
being an ordinary working day there.

**AWT-AC-090: A shift over two days meets two rules.** Given a rule set with a timing rule on
non-working days with the band `0` to `24` and a timing rule outside the company schedule. When
E works from Friday 8 January 2021 `21:00` to Saturday 9 January 2021 `04:00`. Then two lines
exist: `3` hours dated the Friday naming the second rule alone, and `4` hours dated the
Saturday naming both.

**AWT-AC-091: The timing kind defaults.** Given a timing rule created without a timing kind.
Then its timing kind is `work_days`, its band is `0` to `24`, and it therefore matches every
hour of every working day.

---

## Extra hours, dates, time zones and regeneration

**AWT-AC-092: The day of an amount follows the employee's own zone.** Given an employee whose
own zone is `Asia/Tokyo` and whose schedule declares the same zone. When the employee works
from `01:00` to `12:00` on 4 January 2021, which is `10:00` to `21:00` local. Then the
extra-hours line is dated 4 January and the balance is `2`.

**AWT-AC-093: Two far zones agree.** Given the employee of AWT-AC-092 and another whose own
zone and schedule are `Pacific/Honolulu`. When the second works from `17:00` on 4 January to
`04:00` on 5 January, which is `07:00` to `18:00` local. Then each employee has a balance of
`2`.

**AWT-AC-094: Shortfalls in far zones.** Given absence management on, both employees of
AWT-AC-093 and a schedule with a break. When the first works `01:00`–`04:00` — that is
`10:00`–`13:00` local, of which the last hour is the break — and the second works
`17:00`–`20:00`. Then the first has a balance of `−6` and the second of `−5`.

**AWT-AC-095: The schedule's zone wins for the expectation.** Given an employee whose own zone
is `America/New_York` and whose schedule is in `Europe/Brussels`, with rule set R. When the
employee works from `23:30` on 27 May 2024 to `13:30` on 28 May 2024. Then one line of `5`
hours exists, dated 28 May.

**AWT-AC-096: A shift across midnight yields one line per local day.** Given an employee whose
own zone is universal time and whose schedule is in `America/New_York` with a single Sunday
period from `08:00` to `15:00`, and rule set R. When the employee works from 16 November 2025
`10:00` to 17 November 2025 `01:00`. Then two lines exist, of `7` and `1` hours, dated 16 and
17 November; both carry the same start and stop instants, namely the record's own check-in and
check-out. The day split uses the employee's **own** zone, while the seven expected hours of
the Sunday are read from the **schedule's** zone.

**AWT-AC-097: The employee's zone splits the days, the schedule's zone supplies the
expectation.** Given an employee whose own zone is `America/New_York` and whose schedule is in
`Europe/Brussels` with the fixture pattern, rule set R and absence management off. When the
employee works from 30 May 2024 `03:00` to `16:00`, which is `23:00` on 29 May to `12:00` on
30 May in the employee's own zone. Then exactly one line of `4` hours exists, dated 30 May: the
one hour attributed to 29 May falls short of that day's expectation and, absence management
being off, produces nothing, while the twelve hours attributed to 30 May exceed the eight
expected hours by four.

**AWT-AC-098: Regeneration is stable.** Given the situation of AWT-AC-096. When the rule set's
regeneration action is run. Then two lines still exist and their amounts are unchanged, though
their identifiers are different.

**AWT-AC-099: A record ending at midnight is not rebuilt by the next day.** Given E, R, an
attendance from `08:00` to `17:00` on 4 January 2021 and another from `21:00` on 4 January to
`00:00` on 5 January. Then the second reports `3` extra hours. When a further attendance is
created on 5 January from `08:00` to `17:00`. Then the second still reports `3`, because the
scope condition on the check-out is strict.

**AWT-AC-100: A record spanning midnight is recomputed for both days.** Given E and R. When an
attendance runs from 4 January 2021 `22:00` to 5 January 2021 `10:00`. Then it reports `2`
extra hours. When a second attendance is created on 5 January from `14:00` to `18:00`. Then
that second record reports `4`, because the expected quantity of 5 January has already been
consumed by the first two hours of the day.

**AWT-AC-101: Moving a record between employees moves the amount.** Given E and another
employee, both on R, and an attendance of E from `07:00` to `18:00` on 4 January 2021 worth two
extra hours. When an identical attendance is created for the other employee and E's is deleted.
Then the other employee's balance is `2` and E's is `0`.

**AWT-AC-102: Regeneration with several versions and several rule sets.** Given an employee
with a version from 1 March 2020 to 1 April 2020 naming rule set one, and a version from
2 April 2020 naming rule set two, both sets holding a daily rule with a fixed quantity of nine
hours. When the employee works from `07:00` to `18:00` on 4 March 2020 and from `10:00` to
`19:30` on 4 April 2020, and rule set one's regeneration action is run. Then the first record
reports `1` extra hour and the second reports `0.5`: each attendance is evaluated under the
rule set of its own version.

**AWT-AC-103: A change to an exclusion regenerates the affected days.** Given E, R and an
attendance on a day. When a personal exclusion covering part of that day is created, then
moved to another day, then deleted. Then after each operation the lines of every day touched —
the day it left and the day it arrived at — are consistent with the working intervals then in
force.

**AWT-AC-104: Changing a company tolerance regenerates that company.** Given company A with
attendances. When either legacy tolerance amount is written with a different value. Then every
attendance of every employee of company A is regenerated. When it is written with the value it
already has. Then nothing is regenerated.

---

## Approval

**AWT-AC-105: Automatic approval.** Given company A approving automatically, E and R. When E
works from `08:00` to `20:00`. Then one line exists with status `approved`, the record's
validated extra hours are `3` and the balance is `3`.

**AWT-AC-106: Manual approval.** Given the same company switched to "Approved by Manager". When
E works from `08:00` to `20:00`. Then the line's status is `to_approve`, the record's status is
`to_approve`, its validated extra hours are `0` and the balance is `0`. When the record's
approve action is run. Then the status becomes `approved`, the validated extra hours become `3`
and the balance becomes `3`. When the refuse action is then run. Then the balance returns to
`0`.

**AWT-AC-107: A refused shortfall leaves the balance.** Given manager approval, absence
management on, and an attendance from `08:00` to `12:00` producing a line of `−4`. When the
line is approved. Then the record's validated extra hours are `−4`. When the record is refused.
Then they are `0`.

**AWT-AC-108: Approval survives a recomputation of another day.** Given company A approving
automatically, E, R and an attendance on 2 January 2023 from `08:00` to `18:00` whose line was
approved and whose encoded amount was then reduced to `0.5`. When another attendance is created
on 4 January 2023. Then the line of 2 January still has status `approved`, its computed amount
is back to `1.0` and the record's validated extra hours are `0.5`.

**AWT-AC-109: Recomputing the same day forces a new review.** Given manager approval and an
attendance of 5 January 2021 whose line was approved. When a second attendance is created on
**the same day**. Then that day's line reads `to_approve` again. Given a line approved on
4 January 2021 and a weekly rule in the set. When an attendance is created on 8 January 2021.
Then the line of 4 January reads `to_approve` again, because a weekly rule widens the scope to
the whole week.

**AWT-AC-110: Partial approval.** Given manager approval and a line of `1` hour. When its
encoded amount is set to `0.5` and the line is approved. Then the record's validated extra
hours are `0.5` while its extra hours remain `1` and its regular hours are computed from the
`1`.

**AWT-AC-111: An encoded amount above the computed amount is honoured.** Given absence
management on and a record of two worked hours carrying a line of `−6`. When the encoded amount
is set to `10` and the line is approved. Then the extra hours read `−6`, the validated extra
hours read `10` and the regular hours still read `8`.

**AWT-AC-112: The record status follows its lines.** Given a record with two lines. When both
are approved. Then the record's status is `approved`. When one is then refused. Then the
record's status is `to_approve`. When both are refused. Then it is `refused`. When every line
is deleted. Then the status is empty and the approval actions disappear.

**AWT-AC-113: The shared terminal shows the day's total.** Given two lines of `5` hours each
dated today for E, neither approved. When the terminal asks for E's information block. Then
`overtime_today` reads `10`; once both are approved, `total_overtime` also reads `10`, while
`hours_today`, `hours_previously_today` and `last_attendance_worked_hours` all read `0`,
because no attendance was recorded today.

---

## Automation

**AWT-AC-114: Automatic check-out after a forgotten check-out.** Given company A with automatic
check-out and a tolerance of one hour, E with records from `08:00` to `11:00` and `11:00` to
`13:00` and an open record from `14:00`, all on 1 February 2024, and the current instant
1 February 2024 at `23:00`. When the job runs. Then the open record is closed at `19:00`.

**AWT-AC-115: An employee within the allotted time is untouched.** Given the same run and an
employee who checked in at `20:00`, still within the expected time of the local day. Then that
record stays open.

**AWT-AC-116: A flexible employee is never closed automatically.** Given the same run and an
open record of an employee on a flexible schedule, checked in at `12:00`. Then it stays open.

**AWT-AC-117: More than a day elapsed.** Given automatic check-out with a tolerance of one hour
and an open record of E from 30 January 2024 `08:00`, the current instant being 1 February 2024
`23:00`. When the job runs. Then the record is closed at 30 January 2024 `18:00`, inside its own
day.

**AWT-AC-118: Several days elapsed with a smaller excess.** Given automatic check-out with a
tolerance of one hour and an open record of E from 1 February 2024 `14:00`, the current instant
being 5 February 2024 `23:00`. When the job runs. Then the record is closed at 1 February 2024
`23:00`: the provisional close at `23:59:59` gives 9.9997 worked hours, the excess is
9.9997 − ( 8 + 1 − 0 ) = 0.9997, and subtracting it lands one hour earlier.

**AWT-AC-119: The closing instant respects the schedule's zone.** Given an employee whose
schedule declares `Asia/Tokyo`, an open record checked in at `12:00` on 1 February 2024, a
tolerance of one hour and the current instant 1 February 2024 `23:00`. When the job runs. Then
the record is closed at 1 February 2024 `21:00`, which is inside the local day of the check-in.

**AWT-AC-120: The hours already worked are taken from the right local day.** Given an employee
whose schedule declares `Asia/Tokyo` and has **no Friday afternoon and no Friday break**,
records from `06:00` to `07:00` and from `21:00` to `22:00` on 1 February 2024, an open record
from `23:00` on 1 February 2024, a tolerance of one hour and the current instant 2 February
2024 `20:00`. When the job runs. Then the open record is closed at 2 February 2024 `03:00`,
that is four hours after its check-in: four expected hours plus one hour of tolerance minus the
one hour already worked on that Tokyo day, the `06:00`–`07:00` record belonging to the previous
Tokyo day.

**AWT-AC-121: The break is honoured by the automatic closure.** Given automatic check-out with
a tolerance of one hour, a morning record from `08:00` to `12:00` and an open record from
`13:00` on 1 January 2024, the current instant being `22:00`. When the job runs. Then the open
record is closed at `18:00` and the two records together total nine worked hours.

**AWT-AC-122: An absence shortens the expected day.** Given automatic check-out with a
tolerance of `0.1` hour, a personal exclusion from `15:00` to `17:00` and an open record from
`08:00` on 1 January 2024, the current instant being `17:06`. When the job runs. Then the
record is closed at `15:06`, its worked hours are `6.1` and its extra-hours line is `0.1`.

**AWT-AC-123: A two-week schedule is honoured by the automatic closure.** Given a two-week
schedule whose first-week Wednesday holds only an afternoon of four hours, a tolerance of zero
and an open record from `08:00`. When the job runs on a first-week Wednesday at `22:00`. Then
the record is closed at `12:00` with four worked hours. When the same happens on a second-week
Wednesday. Then it is closed at `17:00` with eight worked hours.

**AWT-AC-124: Absence detection creates a negative day.** Given company A with absence
management, an employee whose contract started before yesterday, a schedule expecting eight
hours on that weekday, and no attendance at all yesterday. When the absence job runs. Then a
technical attendance exists for yesterday lasting one second, with both channels `technical`,
and its extra-hours line is approximately `−8`.

**AWT-AC-125: A repaired absence reads zero.** Given the situation of AWT-AC-124. When a real
attendance covering the expected hours of that day is created afterwards. Then the line
attached to the technical attendance reads `0`.

**AWT-AC-126: An unneeded technical attendance is removed.** Given an employee whose schedule
has no working period on the day concerned, or who was on a validated absence all day. When the
absence job runs. Then no technical attendance remains for that employee, because its extra
hours round to zero at three decimal places.

---

## Access and visibility

**AWT-AC-127: An officer may reassign only to employees they approve.** Given an officer who is
the attendance approver of employees X and Y but not of Z, and an attendance of X. When the
officer reassigns it to Y. Then the write succeeds. When the officer reassigns it to Z. Then
the write fails with "Do not have access, user cannot edit the attendances that are not their
own or if they are not the attendance manager of the employee."

**AWT-AC-128: A holder of the officer-for-all group may reassign freely.** Given the same
attendance and a user holding that group. When that user reassigns it to any employee of the
allowed companies. Then the write succeeds.

**AWT-AC-129: Reading extra-hours lines by level.** Given three employees each holding three
approved extra hours: one whose user holds only the own-records group, one whose user is an
officer, and one who names that officer as approver. Then the own-records user reads only their
own line and their own balance of `3`, and reads `0` for anybody else; the officer reads their
own line and the line of the employee they approve, and reads `0` for the third; a holder of
the officer-for-all group reads all three.

**AWT-AC-130: Reading another employee's lines fails for an ordinary user.** Given the same.
When the own-records user asks for the extra-hours line list of another employee. Then the read
is refused by the record rule.

**AWT-AC-131: The manager flag on a line.** Given an officer named as attendance approver of E
and an attendance of E carrying a line. When the officer reads the line. Then its manager flag
is true and the approve and refuse actions are offered.

**AWT-AC-132: The monthly-hours action by level.** Given an ordinary user holding the
own-records group. When that user opens their own public profile. Then the monthly-hours action
returns an action. When that user opens another employee's public profile. Then the action
returns nothing at all. When the same user is given the officer group and is named attendance
approver of that other employee. Then the action returns an action for both profiles.

**AWT-AC-133: Naming an approver grants the officer group.** Given a user who holds no
attendance group. When that user is named attendance approver of an employee. Then the user
holds the officer group. When the approver of that employee is changed to somebody else and the
first user approves nobody else. Then the first user no longer holds the officer group.

**AWT-AC-134: Only the human-resources manager may set the rule set.** Given a user holding the
human-resources user group. When that user opens an employee form. Then the rule-set field is
not present. When the user is given the human-resources manager group. Then the field is
present and writable.

**AWT-AC-135: The employee grouping is widened.** Given two employees, only one of which has an
attendance record, and a reader holding the officer-for-all group. When the attendance list is
grouped by employee. Then only the employee with a record appears. When the reader types the
other employee's name in the search bar. Then that employee appears as an empty group even
though they have no record.

**AWT-AC-136: The terminal routes reject an unknown token.** Given any terminal route. When it
is called with a token that matches no company. Then the page request answers not found, the
employee page route answers with an empty list, every other route answers with an empty
structure, and nothing is written.

**AWT-AC-137: The terminal employee search rejects an arbitrary filter.** Given the employee
page route. When it is called with a condition on any field other than `name` and
`department_id`, or with any operator other than `=` and `ilike`. Then it fails with "Invalid
domain, use 'name' and/or 'department_id' fields with '=' and/or 'ilike' operators."

**AWT-AC-138: The terminal sees only its own company.** Given companies A and B and a
department of company B holding one employee of B and one employee of A. When the terminal page
of company B is opened. Then that department's count reads `1`, and the employee page route of
company B returns only employees of B.

**AWT-AC-139: A wrong personal identification number changes nothing.** Given a company that
demands a number and an employee whose number is `1234`. When the manual-selection route is
called with `9999`. Then the response is an empty structure, no attendance is written, and the
client shows "Wrong Personal Identification Number".

**AWT-AC-140: The menu-bar control follows the selected company.** Given a user with an
employee in company A and another employee in company B. When the menu-bar route is called with
no company selection. Then the employee of the user's default company acts. When it is called
with company B selected first in the client. Then the employee of company B acts.

**AWT-AC-141: Regenerating the terminal key invalidates the old address.** Given a terminal
address in use. When the regenerate action is run by a holder of the officer-for-all group.
Then the previous address answers not found and the new one renders the page.

---

## Reporting

**AWT-AC-142: The regular-hours measure adds up.** Given an attendance of E from `07:00` to
`18:00` with two extra hours. Then its regular hours read `8`. Given a second one from `07:00`
to `08:00` with absence management off. Then its regular hours read `1`. Given a third of two
worked hours with a shortfall of six, absence management being on. Then its regular hours read
`8`.

**AWT-AC-143: The error filter.** Given one record of seventeen worked hours, one open record
checked in two days ago and one ordinary record. When the error filter is applied. Then the
first two appear and the third does not.

**AWT-AC-144: The automatic-closure filter.** Given a record closed by the automatic job and an
ordinary record. When the automatic-closure filter is applied. Then only the first appears, and
it still appears after an officer has corrected its check-out, because the channel is never
rewritten.

**AWT-AC-145: The comparison analysis.** Given an employee with an hourly cost of `30`, eight
presence hours and seven timesheet hours on one day. Then the row of that day reports `7`
timesheet hours, `8` presence hours, a difference of `1`, a timesheet cost of `210`, a presence
cost of `240` and a cost difference of `30`.

---

## Interaction with absence management

**AWT-AC-146: Extra hours can be spent as absence.** Given an employee with four approved extra
hours flagged compensable as time off and an absence kind that deducts extra hours and requires
no allocation. When the employee requests two hours of that kind. Then the request is accepted
and the deductible balance becomes `2`. When the employee requests three more hours. Then the
request is refused with "You do not have enough extra hours to request this leave".

**AWT-AC-147: An approver receives the other message.** Given the same. When a different user
creates the excessive request for that employee. Then it is refused with "The employee does not
have enough extra hours to request this leave."

**AWT-AC-148: An exclusion of kind other does not remove working time.** Given an exclusion of
kind `other` covering a whole working day. When the working intervals of that day are
requested. Then they are unchanged, the day still counts as a working day for a timing rule,
and the expected quantity of a quantity rule is unchanged.

---

## Rounding, precision and boundary edges

**AWT-AC-149: The half-day threshold is inclusive.** Given a schedule whose average hours per
day is `8`, so that the threshold is 8 × 3 ÷ 4 = `6`. When a morning period of exactly six
hours is saved. Then its length in days is `0.5`. When a morning period of `6.01` hours is
saved. Then its length in days is `1`.

**AWT-AC-150: The average hours per day is rounded to two places.** Given a schedule with a
Monday of eight hours, a Tuesday of eight hours and a Wednesday morning of three hours, that is
nineteen hours over three distinct weekdays. Then the total hours per week is `19.00`, the
working days per week is `3` — a day worked at all counts whole — and the average hours per day
is `6.33`, that is 6.333333… rounded once at the end.

**AWT-AC-151: The division uses the unrounded weekly total.** Given the two-week schedule of
AWT-AC-005, whose raw weekly total is seventy-six. When the average per day is computed. Then
it is round( 76 ÷ 2 ÷ 5 , 2 ) = `7.60` exactly. A rebuild that divides the already-rounded
`38.00` obtains the same figure here, but a rebuild that rounds the weekly total to a coarser
step will not.

**AWT-AC-152: A line amount is rounded to four decimal places.** Given the second timing band
of AWT-AC-086, whose stretch runs from `21:00` to `23:59:59.999999`. Then the stored computed
amount is `2.9997`, and the screen renders it as three hours.

**AWT-AC-153: Tolerance comparisons are made at five decimal places.** Given a rule with an
employer tolerance of `0.25` and a day whose balance is `0.250004`. Then the difference rounds
to `0.00000` at five places, the comparison yields equality, the excess does not **exceed** the
tolerance, and no line is produced. Given a balance of `0.2501`. Then the comparison yields
greater, and one line of the whole `0.2501` hours is produced.

**AWT-AC-154: A whole day measures 23.999999 hours.** Given a fully flexible resource. When its
attendance intervals are requested for one whole local day, from `00:00:00.000000` to
`23:59:59.999999`. Then one interval is returned and its length is `23.999999…` hours, not
twenty-four.

**AWT-AC-155: An end hour of twenty-four is the last moment of the same day.** Given a working
period whose end hour is `24` on a Monday. When the working intervals of that Monday are
generated. Then the interval ends at `23:59:59.999999` of that Monday and no part of it falls
on the Tuesday.

**AWT-AC-156: The zero test that removes a technical attendance is made at three decimal
places.** Given a technical attendance of one second whose employee was expected to work
nothing that day. Then its extra hours are `0.0003`, which rounds to `0.000` at three places,
and the record is deleted again.

**AWT-AC-157: The day count is rounded to the nearest thousandth.** Given a duration
measurement that yields 3.4999996 days before rounding. Then the reported day total is `3.5`,
and the reported hour total is **not** rounded.

**AWT-AC-158: The full-time comparison is made at three decimal places.** Given a schedule of
`38.00` hours per week against a reference of `40`. Then it is not full time. Given a schedule
of `39.9996` hours per week against the same reference. Then the difference rounds to `0.000`
and the schedule **is** full time.

---

## Several companies, several schedules and several versions

**AWT-AC-159: A schedule with no company is selectable everywhere.** Given companies A and B
and a Working Schedule whose company is empty. When a resource of company A and a resource of
company B are edited. Then both may choose that schedule, and its full-time reference is not
overwritten by either company's default.

**AWT-AC-160: An employee with no company is visible from every company.** Given an employee
whose company is empty and an attendance of that employee. When a reader whose allowed
companies are A and B lists attendances. Then the record appears, because the company condition
admits an empty employee company.

**AWT-AC-161: Duplicating a company regenerates the terminal key.** Given company A with a
terminal key in use. When company A is duplicated. Then the copy has a different terminal key,
its address answers its own page, and the original address still answers company A's page.

**AWT-AC-162: A change of schedule inside the evaluated window is honoured.** Given an employee
with a version from 1 March 2020 naming an eight-hour schedule and a version from 16 March 2020
naming a six-hour schedule, both with rule set R. When the employee works nine countable hours
on 10 March and nine countable hours on 20 March. Then the first day yields one extra hour and
the second yields three, because each day is measured against the schedule of its own version.

**AWT-AC-163: The company that decides absence management is the rule's, then the employee's.**
Given a rule set whose company is empty and an employee of company A with absence management
on. When the employee works less than expected. Then a shortfall line is produced, the
employee's company having decided. Given the same rule set owned by company B, where absence
management is off, and the same employee. Then no shortfall line is produced.

**AWT-AC-164: Changing the company of a resource resets its schedule.** Given a resource of
company A following company A's default schedule. When its company is changed to B on a form.
Then its schedule becomes company B's default schedule.

**AWT-AC-165: Changing the company of a schedule replaces its periods and its closures.** Given
an unsaved schedule created under company A, holding company A's periods and closures. When its
company is changed to B. Then its periods, its closures, its two-week flag and its time zone
are replaced by copies taken from company B's default schedule.

---

## Lifecycle transitions

**AWT-AC-166: Clearing a check-out removes the lines.** Given a closed attendance of E from
`08:00` to `20:00` carrying one line of `3` hours. When an officer clears the check-out. Then
the record is open, the employee reads `checked_in`, the line has disappeared and the balance
has fallen by three.

**AWT-AC-167: Clearing a check-out is refused while another record is open.** Given the same
closed record and a second, open record of E. When the officer clears the first record's
check-out. Then the write is refused with the message that names the other open record.

**AWT-AC-168: Deleting the newer of two open records.** Given, contrary to the ordinary
guarantee and reachable only by deleting a check-out with elevated rights, an employee with two
open records. When the newer is deleted. Then the employee still reads `checked_in`, because
the older open record becomes the last attendance.

**AWT-AC-169: A checked-in employee is reported present.** Given an employee outside their
working hours whose base presence state is off-hours. When the employee checks in. Then the
presence state reads `present` and the presence icon reads `presence_present`.

**AWT-AC-170: A checked-out employee inside working hours is reported absent.** Given an
employee whose base presence state is off-hours, who is checked out, and whose schedule places
the current instant inside a working interval. Then the presence state reads `absent`.

**AWT-AC-171: Switching a schedule back to one week rebuilds from the company default.** Given
a two-week schedule whose periods were edited by hand. When it is switched back to one week and
the confirmation is accepted. Then every period is deleted, the duration-based flag is false,
and the period list is a copy of the company default schedule's periods.

**AWT-AC-172: Switching a schedule to flexible and back.** Given a fixed schedule with fifteen
periods and averages `40.00` and `8.00`. When the schedule type is set to flexible. Then the
period tab disappears, the two averages stand as they were and are no longer recomputed, and
every interval computation synthesises intervals from them. When the type is set back to fully
fixed. Then the fifteen periods are still there and the averages are recomputed from them.

**AWT-AC-173: Deleting a schedule makes its resources fully flexible.** Given a schedule that
no company uses as its default and a resource that follows it. When the schedule is deleted.
Then the resource's schedule pointer is empty, the resource is fully flexible, and its
attendance intervals cover every queried span.

**AWT-AC-174: A resource in use cannot be deleted.** Given a resource owned by an employee.
When the resource is deleted. Then the deletion is refused by the referential-integrity check.
When the resource is archived instead. Then it disappears from planning searches and every
existing pointer still resolves.

---

## Daylight saving transitions

**AWT-AC-175: A working week that contains a forward transition.** Given a schedule in
`Europe/Brussels` with the fixture pattern, and the week of Monday 30 March 2026, the clocks
having gone forward on Sunday 29 March 2026 at `02:00` local. When the working intervals of
that week are requested. Then they still total `40` hours, because every period is written in
wall-clock time and converted per date; the universal-time instants of that week are one hour
earlier than in the preceding week.

**AWT-AC-176: An attendance that spans a backward transition.** Given a resource in
`Europe/Brussels` with no working period on a Sunday, and the night of Sunday 25 October 2026
when the clocks go back at `03:00` local to `02:00` local. When an employee checks in at
`01:30` local, that is 24 October `23:30`, and out at `03:00` local after the change, that is
25 October `02:00`. Then the elapsed time and the worked hours are `2.50`, although the wall
clock advanced by three and a half hours.

**AWT-AC-177: An attendance that spans a forward transition.** Given the same resource and the
night of Sunday 29 March 2026. When an employee checks in at `00:30` local, that is 28 March
`23:30`, and out at `04:00` local after the change, that is 29 March `02:00`. Then the elapsed
time and the worked hours are `2.50`, although the wall clock advanced by three and a half
hours.

---

## Reconciliation notes

One source version carried a complete scenario file of one hundred and forty-six numbered
scenarios; the other carried its worked examples inside the entity and calculation chapters,
where they remain. The scenarios were renumbered into the single scheme of this file, the
worked examples of the other version were checked against them, and the following points were
resolved.

1. **The renumbering.** The former identifiers ran `AC-001` to `AC-146` with one intercalated
   scenario numbered `AC-096b`. That scenario became `AWT-AC-097`, so the numbering is
   contiguous; every scenario after it moved up by one, and every scenario after the automation
   section's new `AWT-AC-118` moved up by one more. `AWT-AC-096` still denotes the scenario the
   entity file cites: the shift across midnight that splits into one line per local day.
2. **The zone of the fixture.** One version declared the fixture schedule in `Europe/Brussels`
   while writing most of its instants without a zone, which makes the arithmetic of the simple
   scenarios ambiguous by one hour. The fixture schedule is declared in **universal time** here,
   and every scenario that needs two different zones names them; the zone-sensitive scenarios of
   the source version were kept unchanged.
3. **The week type of 17 November 2025.** One version asserted that it is the first week again.
   The week type alternates every seven days from the first day of the calendar, so 10 November
   2025 is the first week, 17 November is the **second** and 24 November is the first again.
   `AWT-AC-016` states the corrected sequence.
4. **The Tokyo automatic check-out.** One version described the schedule only as "without a
   Friday afternoon". The arithmetic reaches `03:00` only when the Friday **break** is absent as
   well; `AWT-AC-120` says so and shows the four-hour result.
5. **Scenarios neither version carried** were added for the rounding and precision edges, for
   several companies and several schedules, for the lifecycle transitions that
   [state-machines.md](state-machines.md) introduced — clearing a check-out, the presence-state
   correction, the schedule-shape switches, the deletion of a schedule — and for the daylight
   saving transitions. They are `AWT-AC-149` to `AWT-AC-177`.
6. **One scenario was added inside a section of the source version.** The automatic check-out
   of a record forgotten for several days, where the excess is smaller than the day itself, was
   exercised by neither version; it is `AWT-AC-118`, and it is the case in which the computed
   instant lands one hour before the end of the check-in's own day rather than at the schedule's
   closing hour.

7. **Every worked example of the other version was checked against these scenarios** and none
   contradicted them; the examples stay where they are, beside the arithmetic they illustrate,
   and are not duplicated here.
