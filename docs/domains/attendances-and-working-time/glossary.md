# Attendances and Working Time — Glossary

Every term this folder uses with a meaning that is narrower, wider or simply different from
ordinary usage. Terms are listed alphabetically. Where a term corresponds to a reproduced
stored value or storage name, that string is given in code font beside it.

**Absence ledger.** The read-only derived table that states, for each employee and each working
day of the last year, the expected hours, the worked hours, the approved absence hours and the
difference. Owned by [Time Off](../time-off/) and specified in
[entities.md, chapter 13](entities.md#13-absence-ledger-shared-with-the-time-off-domain).

**Absence management.** A company setting (`absence_management`). When it is on, a day on which
an employee worked less than expected produces a negative extra-hours line, and the
absence-detection job creates a technical attendance for a working day with no record at all.
It never applies to an employee whose schedule is flexible.

**Approved extra hours.** See *validated extra hours*.

**Attendance.** One presence record of one employee: a check-in instant and an optional
check-out instant, with the evidence of how and from where each side was captured. Open while
the check-out is empty, closed once it is set.

**Attendance approver.** The user named on an employee record (`attendance_manager_id`) who may
read and edit that employee's attendance records and extra-hours lines. Naming a user as
approver grants them the officer group; ceasing to be the approver of anybody removes it again.

**Attendance error.** A record drawn in the error colour: closed with more than sixteen worked
hours, closed with a `technical` check-out channel, or open with a check-in more than a day old.
It is what the "Errors" filter selects.

**Attendance interval.** An interval produced from a Working Schedule's working periods over a
span, carrying the periods that produced it as its payload. Not to be confused with an
attendance **record**.

**Attendance state.** A derived property of an employee (`attendance_state`), either
`checked_in` or `checked_out`, computed from the most recent attendance whose check-in is not in
the future.

**Automatic check-out.** The unattended job, and the company feature it implements, that closes
a forgotten open record once the elapsed time plus the hours already worked that day, less a
tolerance, exceeds the expected working time of the day. The resulting check-out carries its own
capture channel (`auto_check_out`) and is announced in the record's discussion thread.

**Badge identifier.** The value read at a shared terminal in order to identify an employee. It
may come from a printed code or from a radio-frequency token; one field holds both.

**Balance.** Of an employee: the sum of the encoded amounts of that employee's approved
extra-hours lines, over all time. Of a deductible absence kind: see *deductible balance*.

**Break.** A working period whose period kind is `lunch` ("Break"). It carries no working time
and no day fraction, is excluded from the schedule totals and from the working intervals, and is
subtracted from the elapsed time when the worked hours of an attendance are computed.

**Capture channel.** How one side of an attendance was recorded: `kiosk` from a shared terminal,
`systray` from the menu-bar control, `manual` by hand, `technical` by the absence-detection job,
and — on the check-out side only — `auto_check_out` by the automatic check-out job. Recorded by
the operation, never chosen by a person.

**Check-in.** The instant at which a presence starts. Required on every attendance record.

**Check-out.** The instant at which a presence ends. Empty on an open record.

**Closure.** A Working Time Exclusion that names no resource: a public holiday or a company
shutdown. It applies to every resource of the schedule it names, or of the whole company when it
names no schedule.

**Combined pay rate.** The multiplier carried on an extra-hours line, derived from the paid rules
that produced it and from the rule set's combination mode. A rate of zero means the line is not
paid.

**Compensable as time off.** A flag on a rule and on the resulting line, meaning the extra time
is given back as absence entitlement instead of, or in addition to, being paid. Approved
compensable amounts feed the deductible balance.

**Computed amount.** The quantity of extra hours the rules produced for a line (`duration`),
rounded to four decimal places and never edited by a person. Contrast *encoded amount*.

**Day bucket.** The accumulator that holds, for one employee and one local date, the parts of
that employee's attendances which fall on that date. It is the unit a daily quantity rule
measures.

**Day fraction.** The share of a working day that a working period represents, stored as the
period's length in days: zero for a break, one for a full-day period, one half for a period no
longer than three quarters of the schedule's average day, and one above that.

**Decimal hours.** The representation of every duration in this domain: `8.5` means eight hours
and thirty minutes, `0.25` fifteen minutes, `0.1667` ten minutes.

**Deductible balance.** The quantity of banked extra hours an employee may still spend as
absence: the approved compensable amounts, minus the hours of pending or granted requests and
allocations of a deductible absence kind.

**Distinct interval set.** An interval set whose construction keeps two intervals that merely
touch separate, preserving the identity of each source record. Its opposite is the *merging
interval set*.

**Duration-based schedule.** An encoding in which each working period states only a length in
hours and the two clock times are derived by centring that length on twelve o'clock. Break
periods are forbidden on such a schedule.

**Effective zone of an employee.** The zone of the employee's schedule, failing that the
employee's own zone, failing that the zone of the company's default schedule, failing that
universal time. It decides the stored date of an attendance, the worked-hours computation, the
date of a shortfall line and the check-in of a technical attendance. It is **not** the zone the
extra-hours generator uses for its day and week keys, which is the employee's own zone.

**Employer tolerance.** A quantity of hours on a rule that an excess must **exceed** before any
extra time is granted. Once exceeded, the whole excess is granted, not the part above the
tolerance.

**Employee tolerance.** The mirror of the previous term for shortfalls: a shortfall is recorded
only when it exceeds this quantity, and the whole shortfall is then recorded.

**Encoded amount.** The quantity of extra hours actually granted on a line
(`manual_duration`), which an approver may set below or above the computed amount before
approving. It is the figure that reaches the employee's balance.

**Evidence block.** The set of values a device channel records beside an instant: the capture
channel and, when the company enables device and location tracking, the place name, the two
coordinates, the network address and the browser family.

**Expand to whole days.** The interval operation that replaces a set of intervals by one interval
per date touched, from the date's first moment to its last representable moment, with two
corrections that stop an interval brushing midnight from claiming an extra day.

**Expected attendances.** The work intervals an employee is expected to deliver over a span,
obtained by walking the Employee Versions whose contract overlaps the span and unioning each
version's schedule intervals, exclusions subtracted.

**Expected quantity.** The number of hours a quantity rule compares presence against: either the
fixed amount stated on the rule, or the amount read from the employee's own schedule for the
period.

**Extra hours.** Worked time that a rule turns into an approvable quantity. Also called overtime.
A negative quantity is a *shortfall* and shares the same entity.

**Extra-hours line.** One record of the Attendance Overtime Line entity: one employee, one local
day, one quantity, one exact set of rules, one combined rate and one approval status.

**Flexible schedule.** A schedule that fixes no hours and agrees only an average per day and a
total per week. Its working intervals are synthesised around midday, capped per day and per
week.

**Full-time reference.** The number of weekly hours that counts as full time
(`full_time_required_hours`), taken from the company's default schedule, against which a
schedule's work-time rate is measured.

**Fully flexible resource.** A Resource with **no** schedule at all. It is available at every
instant, is never expected to work a particular quantity, never accrues extra hours from a
quantity rule, and has no absence detected for it.

**Half-open interval.** Every interval in this domain is closed at its start and open at its
end. An interval from nine to twelve and an interval from twelve to thirteen do not overlap.

**Instant.** A point on the universal time scale, stored with no zone attached. Contrast
*wall-clock time*.

**Interval.** A triple of a start instant, a stop instant and a payload of the records that
explain it. The universal unit of every computation in this domain.

**Local day.** A calendar date read in a named zone, running from `00:00:00.000000` to
`23:59:59.999999`. Midnight belongs to the day that starts, so a whole day measures 23.999999…
hours.

**Manager flag.** A derived, reader-dependent flag on an attendance and on an extra-hours line
(`is_manager`), true when the reader holds the administrator or officer-for-all group, or holds
the officer group and is the employee's attendance approver. It controls the approval actions
and the editability of the two instants.

**Manually touched day.** A pair of employee and local day whose extra-hours lines carried an
encoded amount different from the computed amount, or were still pending, at the moment a
regeneration began. Lines rebuilt for such a pair are forced back to pending.

**Menu-bar control.** The check-in control shown in every screen of the application to a
signed-in user whose company enables it, letting the user check in and out without opening the
attendance application. Its stored capture channel is `systray`.

**Merging interval set.** An interval set whose construction joins two intervals that touch or
overlap into one, with the union of their payloads.

**Missing hours.** See *shortfall*.

**Non-working day.** For a given employee, a local day on which the schedule places no working
interval once exclusions are removed. Timing rules classify days by this test, which reads the
unusual-day answer of the company's default schedule.

**Officer.** A user who may manage the attendance records of the employees who name them as
attendance approver. Distinguished here from the *officer for all employees*, who may manage
every employee of the allowed companies.

**Open record.** An attendance whose check-out is empty. At most one may exist per employee.

**Overtime.** See *extra hours*.

**Period bucket.** A day bucket or a week bucket; the unit over which a quantity rule totals
presence.

**Personal identification number.** A short secret held on the employee record, demanded at a
shared terminal when the employee is identified by name rather than by badge and the company
requires it.

**Place name.** The human-readable location stored beside a check-in or a check-out, resolved
from the coordinates by an external service, or the literal `Unknown` when that service fails or
is refused.

**Presence intervals.** The parts of an attendance a quantity rule measures: the attendance's own
interval, minus the schedule's break periods, except where a break is itself covered by an
absence, intersected with the period.

**Quantity rule.** A rule that compares the presence of a period — a day or a week — with an
expected quantity and turns the excess into extra time taken from the end of the period, or the
shortfall into a negative line.

**Regeneration.** The wholesale deletion and recreation of every extra-hours line in a scope.
Line identifiers are not stable across it.

**Regular hours.** The worked hours of an attendance minus its extra hours
(`expected_hours`). On a day with a shortfall it equals the expected day, because subtracting a
negative quantity adds it back.

**Resource.** The schedulable unit: a person or a machine, carrying a time zone, a schedule, an
efficiency factor, a company and an activity flag.

**Resource mixin.** The abstract behaviour that gives any entity a Resource of its own and
thereby makes it schedulable.

**Rule set.** A named collection of extra-hours rules with a rate combination mode, assigned to
an employee through the Employee Version effective on the day concerned.

**Schedule picture.** The four interval families assembled per employee before any rule runs:
working periods, break periods, absences, and the stretches during which the employee has no
schedule at all.

**Section marker.** A line of a two-week schedule that carries no time and only separates the
first week from the second, deciding which week each following line belongs to. Its display type
is `line_section`.

**Shared terminal.** A device opened on a company's terminal address, on which employees
identify themselves by badge or by name in order to check in and out. The page is
unauthenticated and is identified only by the token in its address. Its stored capture channel
is `kiosk`.

**Shortfall.** A negative extra-hours line: the employee worked less than expected by more than
the employee tolerance, in a company that manages absences. At most one exists per attendance,
and it is the least severe of those proposed.

**Snap to the schedule.** The operation that moves a pair of instants to the closest working
boundaries: the start to the nearest start of a work interval on its own day, the end to the
nearest end of a work interval within the window.

**Stretch.** A part of an attendance covered by a constant set of rules. One extra-hours line is
created per stretch, per local day and per rule set.

**Technical attendance.** A one-second record created by the absence-detection job for a working
day with no record, whose only purpose is to make the missing hours appear as a negative line.
Both its capture channels are `technical`.

**Terminal key.** The secret segment of a shared terminal's address
(`attendance_kiosk_key`). Regenerating it invalidates every previously distributed link.

**Timing rule.** A rule that treats work performed at a particular moment as extra: on a working
day inside a band of hours, on a non-working day, while the employee is off, or outside a named
schedule.

**Two-week schedule.** A schedule holding two complete weeks that alternate, the week in force
being derived from the calendar date alone.

**Unavailable interval.** The complement of the working intervals between two instants, used by
planning screens to shade time a resource cannot be booked.

**Unusual day.** A local day on which the schedule places no working interval, or, for a flexible
schedule, a day covered by a closure.

**Validated extra hours.** The sum of the **encoded** amounts of the approved extra-hours lines
of an attendance (`validated_overtime_hours`), and, per employee, the balance. Also called
approved extra hours.

**Wall-clock time.** An hour and a minute in a named zone. Working periods are written in
wall-clock time; attendances are stored as instants.

**Week bucket.** The accumulator that holds, for one employee and one week ending on a Sunday,
the parts of that employee's attendances which fall inside that week. It is the unit a weekly
quantity rule measures.

**Week key.** The Sunday that identifies a week bucket. A week runs Monday to Sunday.

**Week type.** Which of the two weeks of a two-week schedule applies on a date, derived from the
parity of the number of whole seven-day blocks since the first day of the calendar. Stored on a
period as `0` "First" or `1` "Second".

**Work-period test.** The test every interval algorithm applies first: a period counts as work
when its kind is not `lunch` and it is not a section marker.

**Work-time rate.** The percentage that a schedule's weekly total represents of the full-time
reference; exactly one hundred when the reference is zero.

**Worked hours.** The elapsed time of an attendance with the schedule's break periods removed,
or the plain elapsed time for a flexible or fully flexible resource.

**Working interval.** A stretch during which a resource is actually expected to work: a working
period of the schedule from which exclusions have been removed.

**Working period.** One line of a Working Schedule: a weekday, a start hour, an end hour, a
period kind, a length in hours and a length in days.

**Working Schedule.** The named weekly pattern of working periods, with a time zone, an average
per day, a total per week, a full-time reference and its own closures. The single source of
truth for when a resource is supposed to work.

**Working Time Exclusion.** A dated span during which work does not happen: a closure when it
names no resource, a personal absence when it names one. Only an exclusion whose time type is
`leave` removes working time.

---

## Reconciliation notes

One source version carried a glossary of the domain's vocabulary; the other defined its terms
inline, at the point of first use, and used different words for several of them. The two were
merged and the following choices were made.

1. **"Shared terminal" rather than the device's product-facing name.** The stored selection value
   is reproduced as `kiosk` wherever it is a value, and the prose says "shared terminal"; the
   same holds for the menu-bar control, whose stored value is `systray`.
2. **"Working Time Exclusion" rather than "resource time off".** One version used the interface
   label. The business name says what the record is; the interface label is stated once, in
   [entities.md](entities.md#5-working-time-exclusion).
3. **"Validated extra hours" and "approved extra hours"** were used interchangeably by the two
   versions. Both are kept, because the stored name is
   `validated_overtime_hours` while the screens say approved; the entry above says they are the
   same figure.
4. **Terms neither version defined** were added because the merged specification uses them:
   attendance interval, day bucket, week bucket, week key, evidence block, expand to whole days,
   expected attendances, half-open interval, local day, manager flag, manually touched day,
   presence intervals, regeneration, schedule picture, stretch and work-period test.
