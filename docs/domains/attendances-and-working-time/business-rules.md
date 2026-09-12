# Attendances and Working Time — Business rules

Every validation, constraint, invariant, permission check and locking rule of the domain,
each with a stable identifier, the condition that makes it fire, the exact text the system
shows when it refuses, and the operations it applies to.

Identifier scheme: `AWT-nnn`. A number is stable within this file; a rule is never
renumbered and a withdrawn number is never reused. Where a rule reproduces a message, the
message is quoted exactly and its placeholders are described in words. Where a rule is
enforced by the database rather than by the application, the rule says so, because a
rebuild must place the check at the same level to obtain the same failure behaviour.

| Range | Subject | Chapter |
|---|---|---|
| AWT-001 … AWT-018 | Resource and the resource mixin | [2](#2-resource-rules) |
| AWT-019 … AWT-039 | Working Schedule and Working Schedule Line | [3](#3-working-schedule-rules) |
| AWT-040 … AWT-053 | Attendance integrity | [4](#4-attendance-integrity-rules) |
| AWT-054 … AWT-059 | Working Time Exclusion | [5](#5-working-time-exclusion-rules) |
| AWT-060 … AWT-089 | Extra hours: generation, tolerances, approval | [6](#6-extra-hours-rules) |
| AWT-090 … AWT-099 | The two unattended jobs | [7](#7-automation-rules) |
| AWT-100 … AWT-114 | Permission and visibility | [8](#8-permission-and-visibility-rules) |
| AWT-115 … AWT-119 | Company, currency and consistency | [9](#9-company-currency-and-consistency-rules) |
| AWT-120 … AWT-126 | Rounding and precision | [10](#10-rounding-and-precision-rules) |
| AWT-127 … AWT-134 | Dates and time zones | [11](#11-date-and-time-zone-rules) |

---

## 1. How to read a rule

Each rule states its **scope** (the entity and the operations it guards), its
**condition** (what makes it fire), its **effect** (what the system does) and, where the
system speaks, the **message**. A message is reproduced character for character, including
its punctuation and any spacing defect; the parts a rebuild substitutes are written in
italics and described.

Three levels of enforcement occur in this domain and they behave differently:

| Level | Behaviour when violated | Example |
|---|---|---|
| Database check | The whole write fails, including every other record in the same batch, and no application message can soften it | `AWT-003`, `AWT-069`, `AWT-081` |
| Validation | The whole write fails with the quoted message; a batch fails as a whole | `AWT-021`, `AWT-041`, `AWT-066` |
| Interactive check | The form refuses the change before it is saved; the record is never written | `AWT-023`, `AWT-020` |

Access checks are a fourth kind: they refuse an operation for a reader rather than for a
value, and they are collected in [chapter 8](#8-permission-and-visibility-rules).

---

## 2. Resource rules

### AWT-001 — A Resource always has a time zone

**Scope.** Resource, creation and write. **Condition.** The time zone (`tz`, "Timezone") is
required and may never be emptied. **Effect.** On creation the value is taken, in order,
from the value supplied explicitly, then from the named user's own zone, then from the
named Working Schedule's zone, then from the zone carried by the request context, then
from the acting user's zone, and finally universal time. Every local-day boundary and every
working period of that resource is read in the resulting zone when no schedule zone
overrides it.

### AWT-002 — A Resource with no Working Schedule is fully flexible

**Scope.** Every interval computation. **Condition.** The schedule pointer (`calendar_id`,
"Working Time") is empty. **Effect.** The resource is available at every instant: its
attendance intervals cover the whole queried span as a single interval, its unavailable
intervals contain only its personal exclusions, its expected quantity is undefined, no
quantity rule produces anything for it, no absence is detected for it, and the
resource-mixin worked-days computation short-circuits to zero days and zero hours. The
consequences are tabulated in
[entities.md, chapter 6.4](entities.md#64-fully-flexible-resources).

### AWT-003 — The efficiency factor is strictly positive

**Scope.** Resource, database check. **Condition.** The efficiency factor
(`time_efficiency`, "Efficiency Factor") is zero or negative. **Message**, reproduced:
"Time efficiency must be strictly positive". **Effect.** The write fails. A factor of one
hundred means nominal speed; two hundred halves an expected duration; fifty doubles it.

### AWT-004 — A Resource follows a Working Schedule of its own company

**Scope.** Resource, the selection offered for the schedule pointer. **Condition.** The
offered schedules are those whose company equals the resource's company. A schedule with no
company is shared and stays selectable from every company. **Effect.** A schedule of
another company cannot be chosen through the interface; a write that names one is not
refused by this rule but is caught by the company-consistency check of the platform.

### AWT-005 — Changing the company resets the schedule; naming a user resets the zone

**Scope.** Resource, interactive change on a form. **Effect.** Choosing another company
replaces the schedule pointer with that company's default Working Schedule. Choosing a user
replaces the time zone with that user's zone. Neither change is applied to records written
without a form.

### AWT-006 — A Resource in use cannot be deleted

**Scope.** Resource, deletion. **Condition.** A record that includes the resource mixin
points at the resource; that pointer restricts deletion. **Effect.** The deletion is
refused by the platform's referential-integrity refusal. Archiving is the supported
retirement. A resource that no host record references may be deleted.

### AWT-007 — Creating a host record creates its Resource

**Scope.** Any entity that includes the resource mixin, creation in bulk. **Condition.** A
set of values does not already name a resource. **Effect.** A Resource is created for it
before the host record is inserted, and its identifier is assigned back to the value set.
The host's resource pointer is required, so no host record can exist without one.

### AWT-008 — A Resource created through the mixin takes the host's name, company, schedule and zone

**Scope.** The creation of AWT-007. **Effect.** The new resource's name is the host's
naming value; its company is the named company or the acting company; its schedule is the
named schedule when there is one; its time zone is the zone removed from the host's own
values when present, and otherwise the named schedule's zone.

### AWT-009 — Duplicating a host duplicates its Resource

**Scope.** Any host of the resource mixin, duplication. **Effect.** The resource is
duplicated as one block with the host. An overridden company or schedule in the duplication
values is applied to the new resource, and the copied host then takes that resource's
company and schedule. The user pointer of a resource is never copied.

### AWT-010 — A Resource write that changes nothing is suppressed

**Scope.** Resource, write, when the acting context requests idempotence and exactly one
resource is being written. **Effect.** Every value equal to the stored value is dropped
from the write; when nothing remains, no write happens at all and no modification timestamp
moves. Records created through the resource mixin always set that context flag, so saving
an unchanged host record does not produce spurious modification tracking on its resource.

### AWT-011 — Bulk creation fills the schedule and the zone

**Scope.** Resource, creation in bulk. **Effect.** For each set of values: when a company is
named and no schedule is, the schedule becomes that company's default; when no time zone is
given, it is taken from the named user's zone, failing that from the named schedule's zone,
failing that from the field default of AWT-001.

### AWT-012 — A Resource is archived, not deleted

**Scope.** Resource. **Effect.** The activity flag (`active`, "Active") defaults to true. A
resource whose flag is false disappears from every default search and from every relation
search unless archived records are explicitly requested, and remains a valid target of
pointers that already exist. Help text of the flag, reproduced: "If the active field is set
to False, it will allow you to hide the resource record without removing it."

### AWT-013 — A human Resource names a user; a material Resource need not

**Scope.** Resource form. **Condition.** The type (`resource_type`, "Type") is `user`
("Human") or `material` ("Material"). **Effect.** On the form the user pointer is required
for a human resource and hidden for a material one. The requirement is a form rule, not a
database constraint: a human resource written without a user is stored.

### AWT-014 — A Resource with no company is visible to every company

**Scope.** Resource, reading. **Effect.** A global record rule limits every read to
resources whose company is among the reader's allowed companies **or** is empty. A resource
with no company is therefore shared.

### AWT-015 — The Resource's own zone is used in exactly two places

**Scope.** Every interval computation. **Effect.** The resource's own zone decides the
answer only when the resource has no schedule, and when an operation is explicitly told to
group resources by their own zone rather than by the schedule's. Everywhere else the
schedule's zone wins (`AWT-128`).

### AWT-016 — The mirrored fields of the mixin are stored and write through

**Scope.** Any host of the resource mixin. **Effect.** The company (`company_id`,
"Company"), the schedule (`resource_calendar_id`, "Working Hours") and the time zone (`tz`,
"Timezone") of the host are mirrors of the resource's values, but they are stored, indexed
and writable; writing one of them writes the same value onto the resource. The company is
precomputed before insertion so that it is available to the record rules of the very first
read.

### AWT-017 — Reading a host record never requires read access to its Resource

**Scope.** Any host of the resource mixin. **Effect.** The resource pointer bypasses the
search-access check, so a user who may not read Resources can still read the records that
own one. This is what lets an ordinary user read an Employee without holding any scheduling
right.

### AWT-018 — Writing the schedule of a Resource that embodies an employee writes the employee too

**Scope.** Resource, write of the schedule pointer. **Condition.** The resource is the
resource of an Employee. **Effect.** The same schedule is written onto the employee, so that
the two never disagree.

---

## 3. Working schedule rules

### AWT-019 — A Working Schedule always has a time zone

**Scope.** Working Schedule, creation and write. **Condition.** The time zone (`tz`,
"Timezone") is required. **Effect.** The default is the zone carried by the acting context,
failing that the acting user's zone, failing that the zone of the built-in administrator
user, failing that universal time. Help text, reproduced: "This field is used in order to
define in which timezone the resources will work." Every start hour and end hour of the
pattern is read in this zone, whatever zone the resources following the schedule declare.

### AWT-020 — A working period never crosses midnight

**Scope.** Working Schedule Line, interactive editing. **Effect.** Changing either hour
clamps both, in this order: the start hour (`hour_from`, "Work from") is reduced to at most
`23.99` and then raised to at least zero; the end hour (`hour_to`, "Work to") is reduced to
at most `24` and then raised to at least zero; the end hour is finally raised to at least
the start hour. A value of exactly twenty-four materialises as `23:59:59.999999` of the same
day and never as midnight of the next. Work that spans midnight is expressed as two periods
on two weekdays. Help text of the start hour, reproduced: "Start and End time of working. A
specific value of 24:00 is interpreted as 23:59:59.999999."

### AWT-021 — Working periods of the same weekday and the same week may not overlap

**Scope.** Working Schedule, validation on the period collection. **Condition.** Two
non-section periods of the same weekday overlap. In two-week mode the test runs once per
week number, so a first-week period and a second-week period never conflict.
**The test.** Each period is mapped to the interval that runs from *weekday index × 24 +
start hour + 0.000001* to *weekday index × 24 + end hour*; the intervals are merged; the
merge must not reduce their count. The microscopic amount added to each start is what makes
two periods that merely touch — one ending at twelve, the next starting at twelve — legal
while genuine overlaps are still detected. **Message**, reproduced: "Attendances can't
overlap.". **Effect.** The save fails as a whole.

### AWT-022 — In two-week mode every period lives under a section marker

**Scope.** Working Schedule, validation on the period collection. **Condition.** The
schedule is in two-week mode, at least one period is a section marker, and the first period
in sequence order is not a section marker. **Message**, reproduced: "In a calendar with 2
weeks mode, all periods need to be in the sections." **Effect.** The save fails.

### AWT-023 — In two-week mode exactly two section markers exist

**Scope.** Working Schedule, interactive editing of the period collection. **Condition.**
The number of section markers whose week number is "first" is not exactly one, or the
number whose week number is "second" is not exactly one. **Message**, reproduced: "You
can't delete section between weeks." **Effect.** The change is refused before the record is
saved. After a legal change, the week number of every other line is reassigned from its
position relative to the two markers.

### AWT-024 — The week in force is derived from the date alone

**Scope.** Every computation on a two-week schedule. **Effect.** The week type is a pure
function of the date and of nothing else — not of the schedule, not of the company, not of
the locale, not of the reader's zone:

```formula
week_type( date ) = floor( ( ordinal_day_number( date ) − 1 ) ÷ 7 )  modulo  2
```

where the ordinal day number counts days from the first day of year one of the proleptic
Gregorian calendar, that day being numbered one. The result is zero, the "first" week, or
one, the "second" week. No counter is stored, so two installations always agree and a
fifty-three-week year causes no drift.

### AWT-025 — A break period carries no working time and no day fraction

**Scope.** Working Schedule Line. **Condition.** The period kind (`day_period`, "Day
Period") is `lunch` ("Break"). **Effect.** Its length in hours is forced to zero and its
length in days is forced to zero. It is excluded from the weekly total, from the daily
average and from the working intervals. It exists so that worked hours can be reduced by it
and so that the day layout can be displayed.

### AWT-026 — A duration-based schedule forbids break periods

**Scope.** Working Schedule Line, validation. **Condition.** The line's period kind is
`lunch` and its schedule has the duration-based flag set. **Message**, reproduced with the
line's name substituted at the front: "*line name* is a break attendance, You should not
have such record on duration based calendar". **Effect.** The save fails. Switching a
schedule to duration-based entry deletes the break periods first, so the ordinary path
never trips this rule.

### AWT-027 — Section markers are inert

**Scope.** Every computation over a period collection. **Condition.** The display type
(`display_type`, "Display Type") is `line_section` ("Section"). **Effect.** The line is
excluded from overlap checking, from every total, from the day and hour counts and from
every interval computation. Both its hours are zero, which is also why the length
computation skips it: that computation runs only for lines whose end hour is non-zero.

### AWT-028 — Averages are derived for fixed schedules and entered for flexible ones

**Scope.** Working Schedule. **Effect.** For a schedule that is not flexible, the average
hours per day (`hours_per_day`, "Average Hour per Day") and the total hours per week
(`hours_per_week`, "Hours per Week") are recomputed on every change of the period
collection, of a start hour, of an end hour, of the two-week flag or of the flexible flag,
by the formulas of [calculations.md, chapter 1](calculations.md#1-the-averages-of-a-working-schedule).
For a flexible schedule they are entered by a person and are never overwritten: they are the
daily budget and the weekly budget.

### AWT-029 — The full-time reference comes from the company's default schedule

**Scope.** Working Schedule. **Effect.** For every schedule that has a company, the
full-time reference (`full_time_required_hours`, "Full Time Equivalent") is recomputed as
the total hours per week of that company's default schedule. A schedule with no company
keeps the value entered by a person. Help text, reproduced: "Number of hours to work on the
company schedule to be considered as fulltime."

### AWT-030 — Switching the week layout rewrites the period collection

**Scope.** Working Schedule, the "switch calendar type" operation. **Effect.** Specified as
a transition in [state-machines.md, chapter 7.2](state-machines.md#72-transition-table).
In one direction two section markers are inserted and every period is duplicated into both
weeks; in the other every period is deleted, the duration-based flag is forced false and
the collection is refilled from the company's default schedule. Both directions ask for
confirmation first.

### AWT-031 — Switching the encoding rewrites the period collection

**Scope.** Working Schedule, the "switch based on duration" operation. **Effect.** Turning
duration-based entry on deletes every break period and leaves the remaining lengths in
place. Turning it off deletes every period and refills the collection from the company's
default schedule, duplicating it into two weeks when the schedule is in two-week mode. Both
directions ask for confirmation first.

### AWT-032 — Changing the company of a schedule replaces its periods and its closures

**Scope.** Working Schedule, write of the company, when the record is new or previously had
a different company. **Effect.** The period collection is replaced by a copy of every period
of the new company's default schedule, the global-closure collection is replaced by a copy
of every global exclusion of that schedule — carrying only the reason, the two instants and
the time type — and the two-week flag and the time zone are copied from it as well.

### AWT-033 — Every company has a default Working Schedule, and it cannot be deleted while in use

**Scope.** Company creation; Working Schedule deletion. **Effect.** A company created
without a default schedule receives one, named `Standard 40 hours/week` and owned by that
company, and that schedule becomes the company default. A schedule that a company points at
as its default cannot be deleted; the deletion is restricted. Every other pointer is emptied
instead: a Resource whose schedule is deleted becomes fully flexible, an exclusion whose
schedule is deleted becomes a closure of its company, and a timing rule whose schedule is
deleted becomes invalid and is refused on its next save by `AWT-068`.

### AWT-034 — Copying a working period carries exactly eight values

**Scope.** Working Schedule duplication, and the copy performed by AWT-032. **Effect.**
Exactly the name, the day of week, the start hour, the end hour, the period kind, the week
number, the display type and the sequence are carried over. The two derived lengths are not
carried over and are recomputed from the copied values.

### AWT-035 — The half-day classification of a period

**Scope.** Working Schedule Line, the length in days (`duration_days`, "Duration (days)").
**Effect.** Zero for a break; one for a period whose kind is `full_day` ("Full Day");
otherwise one half when the length in hours is at most three quarters of the schedule's
average hours per day, and one above that. The value is stored and writable, so an
administrator may override it, and every subsequent day count for that schedule then uses
the overridden value.

### AWT-036 — A flexible schedule ignores its period collection

**Scope.** Every interval computation. **Condition.** The schedule type is `flexible`.
**Effect.** The stored periods are kept but are never read. Intervals are synthesised day by
day, centred on twelve o'clock, capped by the daily budget and by the weekly budget, as
specified in
[working-schedule-algorithms.md, chapter 5](working-schedule-algorithms.md#5-flexible-schedules-and-fully-flexible-resources).
The form hides the period tab entirely.

### AWT-037 — The work-period test

**Scope.** Every interval algorithm and every schedule total. **Effect.** A line counts as
work when its period kind is not `lunch` **and** it is not a section marker. Every
computation applies this test before anything else.

### AWT-038 — Deleting a schedule empties every other pointer

**Scope.** Working Schedule, deletion. **Effect.** The periods are deleted with the schedule
(cascade). Every other pointer is emptied rather than blocked, except the company default of
`AWT-033`. Archiving is therefore the safe way to retire a schedule; the activity flag's
help text, reproduced, is "If the active field is set to false, it will allow you to hide
the Working Time without removing it."

### AWT-039 — The name and the initial periods of a new schedule

**Scope.** Working Schedule, defaults on an empty form. **Effect.** When no name is supplied
but a company is, the name becomes the phrase "Working Hours of " followed by the company
name. When the period collection is requested and empty, it is filled with a copy of the
company default schedule's periods and the two-week flag is copied as well. When the
full-time reference is requested and empty, it is filled from the company default schedule.
When the company has no default, or when copying would import a two-week pattern into a
one-week schedule, the built-in fifteen-line forty-hour pattern of
[entities.md, chapter 3.9](entities.md#39-the-built-in-forty-hour-pattern) is used instead.
Duplicating a schedule appends " (copy)" to its name and does not copy the total hours per
week, which is recomputed.

---

## 4. Attendance integrity rules

The four overlap rules `AWT-041` to `AWT-044` are raised from two validations that run for
every record being written; a batch write therefore fails as a whole when any record in it
conflicts (`AWT-045`).

### AWT-040 — A check-in is required and defaults to the current instant

**Scope.** Attendance, creation. **Condition.** The check-in (`check_in`, "Check In") is
empty. **Effect.** The write fails on the required-value check; the interface names the
field "Check In". On a form the field is pre-filled with the current instant, so the failure
only occurs when a person clears it deliberately or when an integration omits it.

### AWT-041 — A check-out is never earlier than its check-in

**Scope.** Attendance, creation and write, validation on both instants. **Condition.** Both
instants are present and the check-out is strictly earlier than the check-in. **Message**,
reproduced including its internal quotation marks: "\"Check Out\" time cannot be earlier
than \"Check In\" time." **Effect.** The write fails. Equal instants are accepted, so a
record of zero length is legal.

### AWT-042 — A check-in never falls inside an earlier record

**Scope.** Attendance, creation and write, validation on the two instants and the employee.
**Condition.** The latest other record of the same employee whose check-in is at or before
this check-in has a check-out later than this check-in. **Message**, reproduced: "Cannot
create new attendance record for *employee name*, the employee was already checked in on
*date and time*", where *employee name* is the employee's display name and *date and time*
is the offending check-in rendered in the reader's zone and format. **Effect.** The write
fails.

### AWT-043 — An employee has at most one open record

**Scope.** Attendance, creation and write. **Condition.** The record being written has no
check-out **and** another record of the same employee has none either. **Message**,
reproduced: "Cannot create new attendance record for *employee name*, the employee hasn't
checked out since *date and time*", where *date and time* is the check-in of the other open
record. **Effect.** The write fails.

### AWT-044 — A record never swallows another record

**Scope.** Attendance, creation and write. **Condition.** The record being written has a
check-out, and the latest other record whose check-in is strictly earlier than that
check-out is not the record already examined by `AWT-042`. **Message**, reproduced: "Cannot
create new attendance record for *employee name*, the employee was already checked in on
*date and time*", naming the conflicting record's check-in. **Effect.** The write fails.

### AWT-045 — A batch write fails as a whole

**Scope.** Attendance, creation and write of several records at once. **Effect.** The
validations of `AWT-041` to `AWT-044` run for every record in the set; the first failure
aborts the entire operation and no record of the batch is written. An importer must
therefore repair the whole batch, not only the offending row.

### AWT-046 — Reassigning an attendance to another employee is restricted

**Scope.** Attendance, write of the employee. **Condition.** The new employee is not one of
the writer's own employees, the writer does not hold the attendance administrator group,
and the writer is not the attendance approver of the new employee. **Message**, reproduced:
"Do not have access, user cannot edit the attendances that are not their own or if they are
not the attendance manager of the employee." **Effect.** The write fails before anything is
stored.

### AWT-047 — An attendance cannot be duplicated

**Scope.** Attendance, duplication. **Condition.** Always. **Message**, reproduced: "You
cannot duplicate an attendance." **Effect.** The operation fails. A duplicate would violate
`AWT-042` by construction and would double-count worked time.

### AWT-048 — A check-out with no matching check-in is refused

**Scope.** The change-state operation invoked by every capture channel. **Condition.** The
employee's attendance state reads `checked_in` but no record of that employee without a
check-out can be found. **Message**, reproduced: "Cannot perform check out on *employee
name*, could not find corresponding check in. Your attendances have probably been modified
manually by human resources." **Effect.** Nothing is written. The situation arises when an
officer deletes an open record between the reading of the state and the check-out.

### AWT-049 — The day of an attendance is the local day of its check-in

**Scope.** Attendance, the stored date (`date`, "Date"). **Effect.** The date is required,
indexed and precomputed before insertion, and holds the calendar date of the check-in read
in the employee's **effective** zone: the zone of the employee's schedule, failing that the
employee's own zone, failing that the zone of the company's default schedule, failing that
universal time. Storing it lets a grouping by day proceed without re-deriving a zone. When
either the employee or the check-in is missing at precompute time, today's date is used; the
situation cannot persist after creation, because both are required.

### AWT-050 — Capture channels are recorded, never chosen

**Scope.** Attendance, both channel fields. **Effect.** The channel of each side is written
by the operation that wrote the instant, as tabulated in
[state-machines.md, chapter 6](state-machines.md#6-the-capture-channel-of-each-side-of-an-attendance).
Both fields are read-only on every screen, and neither is ever cleared.

### AWT-051 — Location evidence is captured only when the company enables it

**Scope.** Every check-in and check-out through a device channel. **Condition.** The
employee's company has device and location tracking switched off. **Effect.** No
coordinates, place name, network address or browser family are recorded and the evidence
blocks are hidden on the form. When tracking is on and the place lookup fails or is refused,
the place is recorded as the literal `Unknown` and the coordinates supplied by the client
are used, or, in their absence, those derived from the network address.

### AWT-052 — A record is flagged as an error when it is implausible

**Scope.** Attendance, the display colour (`color`, "Color"). **Effect.** The colour is one
when the record is closed and either its worked hours exceed sixteen or its check-out
channel is `technical`, and also when the record is open and its check-in is more than one
day before the current instant. An open record younger than a day is coloured ten;
everything else is coloured zero. Colour one is rendered as an error and colour ten as a
success. A record in colour one is what the screens call an attendance error, and it is what
the "Errors" filter selects.

### AWT-053 — Archiving an employee closes the open record

**Scope.** Employee, archiving. **Effect.** Every open attendance of the archived employees
is closed at the current instant, with elevated rights, so that a human-resources user who
holds no attendance right can archive an employee without leaving a dangling open record.
The capture channel of the check-out is unchanged and therefore stays at its stored value.

---

## 5. Working Time Exclusion rules

### AWT-054 — An exclusion starts before it ends

**Scope.** Working Time Exclusion, creation and write, validation on the two instants.
**Condition.** The start instant is later than the end instant. **Message**, reproduced:
"The start date of the time off must be earlier than the end date." **Effect.** The write
fails. When the end instant is empty or not strictly later than the start, it is recomputed
before the check runs: the start is read in the acting user's zone — failing that in the
zone of the exclusion's company's default schedule, failing that universal time — its time
of day is replaced by `23:59:59`, and the result is converted back. A valid, later end
instant entered by a person is never overwritten.

### AWT-055 — Two global closures of the same schedule may not overlap

**Scope.** Working Time Exclusion, validation contributed by the absence domain.
**Condition.** Two exclusions with no resource, attached to the same schedule, overlap in
time. **Message**, reproduced: "Two public holidays cannot overlap each other for the same
working hours." **Effect.** The write fails.

### AWT-056 — An exclusion, its schedule and its company agree

**Scope.** Working Time Exclusion. **Effect.** The company is derived from the named
schedule and falls back to the acting company when no schedule is named. The selectable
schedules are those of the exclusion's company or those with no company. When the named
schedule's company is neither empty nor equal to the exclusion's company, the platform's
standard company-consistency refusal applies; it is specified in
[../../overview/security-model.md](../../overview/security-model.md).

### AWT-057 — A global closure applies only inside its company

**Scope.** Every interval computation. **Condition.** The exclusion names no resource.
**Effect.** It removes time from a resource only when the company of the resource equals the
company of the exclusion. When the exclusion names no schedule either, it applies to every
schedule of that company; when it names one, it applies to that schedule only. A closure
declared by one company therefore never shortens another company's working day, and never
turns another company's Tuesday into a non-working day.

### AWT-058 — Only an exclusion whose time type is `leave` removes working time

**Scope.** Every interval computation. **Effect.** The default filter selects exclusions
whose time type is `leave` ("Time Off"). An exclusion of type `other` ("Other") stays inside
the working intervals; it exists so that training, briefings and other paid but
non-productive time can be marked without shortening the working day. Help text of the
field, reproduced: "Whether this should be computed as a time off or as work time (eg:
formation)".

### AWT-059 — An absence of a flexible resource blocks whole days

**Scope.** The leave-interval algorithm. **Condition.** The resource concerned is flexible
or fully flexible. **Effect.** Its exclusion intervals are widened to run from local
`00:00:00.000000` of the first day to local `23:59:59.999999` of the last day, because a
flexible resource has no fixed hours from which a partial absence could be subtracted.

---

## 6. Extra-hours rules

### AWT-060 — Regeneration forces a new review of a day a person has touched

**Scope.** Every regeneration of extra hours. **Effect.** Before the lines of a scope are
deleted, the pairs of employee and day are remembered for every line whose encoded amount
(`manual_duration`, "Extra Hours (encoded)") differs from its computed amount (`duration`,
"Extra Hours"), and for every line whose status is still `to_approve`. Every line rebuilt
for one of those pairs is forced to `to_approve`, whatever the company's validation setting
says, because the amount has changed underneath a human decision. Lines of untouched days
are rebuilt with the status the company setting implies, so a day that was automatically
approved stays approved.

### AWT-061 — A regeneration deletes and recreates; line identifiers are not stable

**Scope.** Every regeneration. **Effect.** The lines in scope are deleted and new lines are
created; they are never updated in place. Line identifiers are therefore **not** stable
across regenerations, and a downstream consumer must join on the employee, the day and the
two instants rather than on the identifier. A regeneration is triggered by: the creation of
an attendance; a write that changes an attendance's employee, check-in or check-out; the
deletion of an attendance; the creation, movement or deletion of a Working Time Exclusion; a
change to either of the two company tolerance amounts; and the regeneration action on an
Overtime Ruleset.

### AWT-062 — The recomputation scope widens to whole weeks when a weekly rule exists

**Scope.** The construction of the recomputation window. **Condition.** Any rule of any rule
set applicable to the attendances at hand uses the weekly period. **Effect.** The local
range runs from the Monday on or before the earliest local day to the Sunday on or after the
latest local day. Otherwise it is exactly the local days the attendances touch. A weekly rule
makes neighbouring days interdependent, so a narrower scope would leave stale amounts.

### AWT-063 — A record ending exactly at the start of the recomputed range is out of scope

**Scope.** The scope condition of a regeneration. **Effect.** The condition selects
attendances whose check-in is at or before the last moment of the last local day and whose
check-out is **strictly after** the first moment of the first local day. An attendance that
ends exactly at midnight is therefore not rebuilt when the following day is recomputed, and
keeps the amount it earned on its own day.

### AWT-064 — Only closed attendances produce extra hours

**Scope.** Every step of the generation. **Effect.** Records without a check-out are ignored
everywhere: an open record never contributes to a period total, never carries a line and
never appears in a scope. It follows that an employee who is still at work shows no extra
hours for the current stretch.

### AWT-065 — The applicable rule set is the one named by the effective Employee Version

**Scope.** Every generation. **Effect.** For each attendance, the Employee Version in force
at the local check-in names the rule set. An attendance whose effective version names no
rule set produces no line at all. An employee whose versions name different rule sets over
time is evaluated with each rule set on its own period, and the attendances are grouped by
rule set before the generation runs.

### AWT-066 — A quantity rule needs an expected quantity

**Scope.** Overtime Rule, creation and write of the family, the fixed expected hours or the
period. **Condition.** The family (`base_off`, "Based Off") is `quantity`, the expectation is
not taken from the employee's schedule, and the fixed expected hours is zero or empty.
**Message**, reproduced with the rule's name substituted: "Rule '*rule name*' is based off
quantity, but the usual amount of work hours is not specified". **Effect.** The write fails.

### AWT-067 — A quantity rule needs a period

**Scope.** Overtime Rule, the same triggers. **Condition.** The family is `quantity` and the
period (`quantity_period`, "Quantity Period") is empty. **Message**, reproduced: "Rule
'*rule name*' is based off quantity, but the period is not specified". **Effect.** The write
fails. The field defaults to `day` ("Day"), so the failure only occurs when it is cleared
deliberately.

### AWT-068 — A named-schedule timing rule needs a schedule, and that schedule must not be flexible

**Scope.** Overtime Rule, creation and write of the family, the timing kind or the named
schedule. **Condition.** The family is `timing`, the timing kind (`timing_type`) is
`schedule` ("Outside of a specific schedule"), and no schedule is named. **Message**,
reproduced: "Rule '*rule name*' is based off timing, but the work schedule is not
specified". **Effect.** The write fails. The selection offered for the schedule pointer
excludes flexible schedules, because a flexible schedule has no fixed hours to be outside
of.

### AWT-069 — Timing bounds are hours of the day

**Scope.** Overtime Rule, database checks. **Conditions and messages**, both reproduced:
the start hour must satisfy *at least zero and strictly less than twenty-four*, otherwise
"Timing Start is an hour of the day"; the stop hour must satisfy *at least zero and at most
twenty-four*, otherwise "Timing Stop is an hour of the day". **Effect.** The write fails at
the database level, so the whole transaction is lost.

### AWT-070 — A timing window whose start exceeds its stop wraps around midnight

**Scope.** Timing rules of the kinds `work_days` and `non_work_days`. **Effect.** When the
start hour is not greater than the stop hour, the band on a date runs from that date at the
lower value to that date at the higher value. When the start hour is greater than the stop
hour, the band is the inversion of that interval over the whole date: from the date's first
moment to the lower value, together with the higher value to the date's last representable
moment. A window from fourteen to five therefore matches the small hours of the morning and
the late afternoon and evening of the same calendar day, each day being treated
independently.

### AWT-071 — Break time is never extra time

**Scope.** Quantity rules. **Effect.** Before a quantity rule measures a stretch of
presence, the schedule's break periods are removed from it — **except** where the break is
itself covered by an absence, because on an absent day there is no break to take. An
employee who works straight through the break therefore gains no extra hours from it. The
subtraction is written *presence = ( attendance interval minus ( break minus absence ) ) and
period*.

### AWT-072 — The expected quantity may come from the employee's schedule

**Scope.** Quantity rules. **Effect.** When the flag `expected_hours_from_contract` ("Hours
from employee schedule") is false, the expected quantity is the fixed number of hours stated
on the rule. When it is true and the employee's version is not flexible, the expected
quantity is the total length of *( the schedule's working periods minus the absences ) and
the period*. When it is true and the version is flexible, it is the total length of the
employee's expected attendances over the period, computed from the first moment of the
period's first date to the last representable moment of its last date and treated as
universal time.

### AWT-073 — Extra time is taken from the end of the period

**Scope.** Quantity rules. **Effect.** Once the excess of a period is known, it is attributed
by consuming the expected quantity from the earliest presence interval forward and marking
everything after that point as extra. The employee's earliest hours are therefore always the
regular ones, whichever attendance they belong to.

### AWT-074 — The employer tolerance is a threshold, not a deduction

**Scope.** Quantity rules and timing rules. **Effect.** No extra time at all is granted
unless the excess **exceeds** the employer tolerance, the comparison being made at five
decimal places. Once it does, the **whole** excess is granted, not the part above the
tolerance. An excess exactly equal to the tolerance is absorbed. For a timing rule the same
threshold applies to the total time that rule matches within one attendance, and the whole
of that time is kept or none of it.

### AWT-075 — The employee tolerance is a threshold on shortfalls

**Scope.** Quantity rules. **Effect.** A shortfall is recorded only when the company that
owns the rule — failing that the employee's company — has absence management enabled **and**
the balance is strictly below the negative of the employee tolerance, compared at five
decimal places. The amount recorded is then the whole shortfall, not the part beyond the
tolerance. A shortfall exactly equal to the tolerance produces nothing.

### AWT-076 — At most one shortfall per attendance, and it is the least severe

**Scope.** The emission of negative lines. **Effect.** When several quantity rules, of the
same period or of different periods, all report a shortfall for the same attendance, only
the **greatest** amount survives — the least negative, that is the smallest shortfall.
Shortfalls are never summed. An employee short by three hours against one rule and by five
against another is recorded as short by three.

### AWT-077 — A shortfall is attached to the last attendance of its period

**Scope.** The emission of negative lines. **Effect.** The attendance with the latest
check-out inside the period carries the negative line, and the line is dated with the
check-in of that attendance read in the employee's **effective** zone. When the period
contains no presence interval at all, nothing is produced.

### AWT-078 — Extra time is split at every change in the applicable rule combination

**Scope.** The emission of positive lines. **Effect.** After every rule has proposed its
intervals for an attendance, the intervals are decomposed until each resulting stretch is
covered by a constant set of rules; one line is created per stretch, per local day and per
rule set. A stretch covered by a daily rule and a weekly rule at the same time therefore
produces a line distinct from the neighbouring stretch covered by the weekly rule alone, and
two rules with the same expectation produce one line rather than two.

### AWT-079 — A line records the attendance, not the stretch

**Scope.** Attendance Overtime Line. **Effect.** The start instant (`time_start`, "Start")
and the stop instant (`time_stop`, "Stop") of a line are the check-in and the check-out of
the attendance that carries it, never the boundaries of the stretch. This is the join key
between a line and its attendance, and it is why an attendance spanning midnight owns
several lines that all carry the same pair of instants but different days.

### AWT-080 — Line amounts are rounded to four decimal places

**Scope.** The emission of every positive line. **Effect.** The computed amount is the sum
of the stretch lengths in hours, rounded to four decimal places, half away from zero. The
encoded amount is set to the same value at creation.

### AWT-081 — The stop instant of a line is strictly after its start instant

**Scope.** Attendance Overtime Line, database check. **Condition.** The stop instant is not
strictly later than the start instant. **Message**, reproduced: "Starting time should be
before end time." **Effect.** The write fails at the database level. A generated line always
satisfies the check, because a zero-length attendance produces no presence and therefore no
line.

### AWT-082 — Overlapping lines are legitimate

**Scope.** Attendance Overtime Line. **Effect.** No constraint prevents two lines of the
same employee from overlapping in time, and none should be added: two different rules can
attribute different quantities to overlapping spans of the same attendance, and a daily rule
and a weekly rule can both fire on the same attendance.

### AWT-083 — The combined pay rate is derived from the paid rules only

**Scope.** The emission of every line. **Effect.** When no rule that produced the stretch is
flagged paid, the rate is zero and the line is not paid at all. In "Maximum Rate" mode the
rate is the highest rate among the paid rules, ties broken by the higher sequence. In "Sum
of all rates" mode the rate is one plus the sum of each paid rule's rate minus one, except
that a paid rule which also gives the time back as time off contributes its whole rate
instead of its rate minus one. The line is flagged compensable as time off when **any**
contributing rule is, paid or not.

### AWT-084 — The status of a new line follows the company setting and is computed only while empty

**Scope.** Attendance Overtime Line, the status. **Effect.** The status is computed as
`to_approve` when the employee's company has extra-hours validation set to "Approved by
Manager" and as `approved` otherwise — but the computation runs **only while the field is
empty**, so an approval or a refusal is never overwritten by a later recomputation. It is
lost only when the line itself is deleted and recreated, which is what `AWT-060` guards
against.

### AWT-085 — The encoded amount is what counts towards the balance

**Scope.** Attendance Overtime Line, the two amounts. **Effect.** The computed amount is
what the rules produced and is never edited by a person. The encoded amount is copied from
it whenever it changes and may then be overwritten by an approver, above or below the
computed amount. The employee's balance, the attendance's validated extra hours and the
deductible balance of the absence domain all use the **encoded** amount; the attendance's
extra hours and regular hours use the **computed** amount.

### AWT-086 — A fully flexible employee never accrues extra hours from a quantity rule

**Scope.** Quantity rules. **Condition.** Subtracting the employee's fully-flexible periods
from the rule's period leaves nothing. **Effect.** The rule is skipped entirely for that
employee and that period. An employee with no schedule at all therefore has neither excess
nor shortfall, however long the presence.

### AWT-087 — Shortfalls exist only under absence management

**Scope.** The emission of negative lines. **Effect.** A negative line is produced only when
the company that owns the rule, failing that the employee's company, has absence management
switched on. With the setting off, a day worked short produces no line at all and the
record's regular hours equal its worked hours.

### AWT-088 — The extra-hours status of an attendance is derived from its lines

**Scope.** Attendance. **Effect.** Empty when no line is linked; `approved` when every
linked line is approved; `refused` when every linked line is refused; `to_approve` in every
other case, including a mixture of approved and refused. The transitions are in
[state-machines.md, chapter 3](state-machines.md#3-the-extra-hours-status-of-an-attendance).

### AWT-089 — Approval and refusal act on every line of the same employee and day

**Scope.** The approve and refuse operations on an Attendance. **Effect.** They set the
status of **every** line linked to that record, not only the line the reader is looking at.
The button help texts are reproduced as "Approve all overtimes for the same employee and
date" and "Refuse all overtimes for the same employee and date". To grant part of an amount,
the encoded amount of a single line is edited first and that line alone is then approved.

---

## 7. Automation rules

### AWT-090 — Automatic check-out fires only past the expected day plus the tolerance

**Scope.** The automatic check-out job. **Condition.** For an open attendance whose
employee's company enables automatic check-out and whose employee's schedule is not
flexible, the job acts only when

```formula
elapsed since the check-in + hours already worked that local day − tolerance  >  expected hours of that local day
```

where every quantity is in decimal hours, the local day is read in the zone of the version
covering the attendance's date, and the expected hours already exclude breaks and approved
absences. **Effect.** When the test fails, the record stays open and nothing at all is
written.

### AWT-091 — The computed check-out never leaves the day of the check-in and never precedes the check-in

**Scope.** The automatic check-out job, when `AWT-090` fires. **Effect.** The check-out is
written as the later of *the last second of the check-in's local day minus the excess* and
*the check-in plus one second*, where the excess is the worked hours of the provisionally
closed record minus *( expected + tolerance − hours already worked that day )*. An
attendance forgotten for several days is therefore truncated back into its own first day.
The one exception is a negative excess, which pushes the computed instant forward by at most
a couple of seconds into the next local day; that case is worked through in
[calculations.md, chapter 12.3](calculations.md#123-the-test-and-the-truncation).

### AWT-092 — An automatic check-out is announced in the record's thread

**Scope.** The automatic check-out job. **Effect.** The check-out channel becomes
`auto_check_out` and a note is posted in the record's discussion thread. **Message**,
reproduced: "This attendance was automatically checked out because the employee exceeded
the allowed time for their scheduled work hours."

### AWT-093 — A flexible employee is never closed automatically

**Scope.** The selection step of the automatic check-out job. **Effect.** Open records are
considered only for employees whose company enables the feature and whose schedule is not
flexible. **Compatibility finding.** A *fully flexible* employee has no schedule at all, so
the flexible-schedule test does not exclude them; their expected hours are nevertheless
empty, so the test of `AWT-090` fires and the record is truncated to a one-second
attendance. A corrected behaviour would exclude fully flexible employees from the selection,
as the rule for flexible schedules already does. A rebuild that adopts the correction must
document the divergence, because the observed behaviour produces a one-second record.

### AWT-094 — Absence detection covers yesterday only

**Scope.** The absence-detection job. **Effect.** Each run considers the single day before
the current day, evaluated on the unattended process's own clock and not in any employee's
zone. Employees are skipped when they already have any extra-hours line dated that day, when
their schedule is flexible, when their company has not enabled absence management, or when
their current version's contract has not started by that day.

### AWT-095 — A detected absence is a one-second technical attendance

**Scope.** The absence-detection job. **Effect.** The record runs from the first moment of
that day, read in the employee's effective zone and converted to the universal scale, to one
second later; both capture channels are `technical`. Its only purpose is to force the day
through the extra-hours generator, which then reports the whole expected day as a shortfall.

### AWT-096 — A technical attendance that yields no shortfall is removed

**Scope.** The absence-detection job, after the generation has run. **Effect.** Every
technical attendance whose resulting extra hours are zero at three decimal places is deleted
again, so an employee who was not in fact expected to work yesterday leaves no trace.

### AWT-097 — A surviving technical attendance is announced in its thread

**Scope.** The absence-detection job. **Effect.** A note is posted on each surviving record.
**Message**, reproduced: "This attendance was automatically created to cover an unjustified
absence on that day."

### AWT-098 — Both jobs run every four hours and are idempotent

**Scope.** The two unattended jobs. **Effect.** Each is scheduled at an interval of four
hours. A second run inside the same interval finds no candidate, because the first run
either closed the record or created the technical attendance, and an employee who already
has a line dated yesterday is skipped. Their behaviour depends only on the current instant,
the company settings and the stored records, so they tolerate being run at any hour.

### AWT-099 — Both jobs run with elevated rights

**Scope.** The two unattended jobs. **Effect.** They write attendance records of employees
they are not the approver of, so they run outside the record rules of
[chapter 8](#8-permission-and-visibility-rules). A rebuild must grant them the same
privilege or they will silently skip most employees.

---

## 8. Permission and visibility rules

### AWT-100 — Four access levels govern attendance

**Scope.** Every operation on an Attendance, an Attendance Overtime Line, an Overtime Rule
and an Overtime Ruleset.

| Level | Group | Attendance records | Extra-hours lines | Rules and rule sets | Configuration |
|---|---|---|---|---|---|
| Own records | "User: Read his own attendances", implied by the base internal-user group | read own only | read own only | none | none |
| Officer | "Officer: Manage attendances" | create, read, update, delete for the employees who name the user as attendance approver | the same set | read rules | none |
| Officer for all | "Officer: Manage all attendances", which implies the officer group | create, read, update, delete for every employee of the allowed companies | all | read rules | open the shared terminal, regenerate its key, load the onboarding scenario |
| Administrator | "Administrator", which implies the officer-for-all group | as above | as above | create, read, update, delete on rules and rule sets | the settings page and the rule-set menu |

### AWT-101 — Record-level filters on attendances

**Scope.** Attendance, every operation. **Effect.** Four record rules apply, on top of the
coarse access rights:

1. A **global** company condition: the employee has no company, or the employee's company is
   among the reader's allowed companies. It applies to everybody and to every operation.
2. The officer-for-all group sees every record, unconditionally, with full rights.
3. The officer group sees the records whose employee names the reader as attendance
   approver, with create, read, update and delete. Being the employee is not by itself
   enough: both branches of the condition require the approver to be the reader.
4. The own-records group reads the records whose employee's user is the reader, and may not
   create, write or delete through that rule.

### AWT-102 — Record-level filters on extra-hours lines

**Scope.** Attendance Overtime Line, every operation. **Effect.** The same shape: a global
company condition on the employee's company; unconditional full rights for the
officer-for-all group; the approver condition for the officer group, with all four
operations; and a read-only own-records condition for the own-records group. A reader who
may not read another employee's lines sees a balance of zero for that employee rather than
an error.

### AWT-103 — The approval buttons require the manager flag

**Scope.** The approve and refuse actions on an Attendance and on an Attendance Overtime
Line. **Effect.** The buttons are offered only when the derived flag is true: the reader
holds the administrator group or the officer-for-all group, or holds the officer group and
is the attendance approver of the employee. The same flag controls whether the two instants
of an attendance are editable on the form.

### AWT-104 — Naming an attendance approver grants the officer group

**Scope.** Employee, creation and write of the attendance approver. **Effect.** The named
user is added to the officer group, with elevated rights, both when the employee is created
with an approver and when the approver is written afterwards.

### AWT-105 — Losing every approval relationship removes the officer group

**Scope.** Employee, write of the attendance approver. **Effect.** After the write, every
user who was the previous approver and is no longer the attendance approver of any employee
is removed from the officer group. A user who holds the group for another reason keeps their
other groups.

### AWT-106 — Rule sets are readable across companies but writable only by administrators

**Scope.** Overtime Ruleset. **Effect.** A global record rule grants **read** to rule sets
whose company is among the reader's allowed companies or is empty, and forbids creating,
writing and deleting through that rule. A second rule grants create, read, write and delete
on the same set to the attendance administrator group only.

### AWT-107 — Only the human-resources manager group may set the rule set on a version

**Scope.** Employee Version, the rule-set pointer. **Effect.** The field is visible and
writable only to that group; the same group may read Overtime Rules and Overtime Rulesets so
that it can choose one. The selection is restricted to rule sets with no country or with a
country among the acting companies' countries.

### AWT-108 — Working schedules and their periods are read widely and written narrowly

**Scope.** Working Schedule and Working Schedule Line. **Effect.** Every internal user may
read them. Writing is granted to the platform settings group and, through the grants
contributed by other domains, to the human-resources officer group and — for the period
lines — to the manufacturing user and manufacturing manager groups. The project user group
holds read access only. The complete matrix is in
[configuration.md, chapter 8](configuration.md#8-access-rights-by-entity).

### AWT-109 — Working time exclusions follow a split rule

**Scope.** Working Time Exclusion. **Effect.** Four record rules apply: a global company
condition (the company is among the reader's allowed companies or is empty); every internal
user may **read** records that have no resource and records whose resource has no user or
has the reader as user; every internal user may **create, write and delete** records whose
resource is set and whose resource has no user or has the reader as user; and only the
platform settings group may create, write or delete records that have no resource. In words:
anyone may see the company's closures, anyone may manage their own absences, and only an
administrator may declare a closure.

### AWT-110 — The shared-terminal endpoints authenticate by company token only

**Scope.** Every endpoint of the shared terminal. **Effect.** The company is resolved from
the token in the request; when no company matches, the endpoint answers with an empty
structure — or, for the page request, with a not-found response — and writes nothing.
Employee reads and attendance writes are then performed with elevated rights, restricted to
employees of that company. A caller cannot tell an unknown token from an unknown employee
from a wrong personal identification number, and that indistinguishability is deliberate.

### AWT-111 — The employee search of the shared terminal accepts a restricted filter only

**Scope.** The employee page endpoint of the shared terminal. **Condition.** The supplied
filter names any field other than the name and the department, or uses any operator other
than *equals* and *contains*. **Message**, reproduced: "Invalid domain, use 'name' and/or
'department_id' fields with '=' and/or 'ilike' operators." **Effect.** The call fails. This
is what prevents an unauthenticated caller from mining the employee directory.

### AWT-112 — Regenerating the terminal key invalidates every distributed link

**Scope.** The regenerate operation on the company or on the settings page. **Effect.** The
key is replaced by a freshly generated random value; every previously distributed terminal
address answers not found from that moment. The operation executes only for a reader holding
the officer-for-all group.

### AWT-113 — The employee grouping of the attendance list is widened deliberately

**Scope.** Attendance, grouping by employee. **Effect.** The group headers are completed
with employees of the allowed companies that have no record in the current selection: all of
them for a reader holding the officer-for-all group, and only those who name the reader as
approver otherwise. When the reader has typed a name in the search bar, the employees
matching that name are **added** to the groups rather than intersected with them, so an
employee with no attendance at all still appears when searched for by name.

### AWT-114 — The monthly-hours action returns nothing when the reader may not see the employee

**Scope.** The "Monthly Hours" action on an employee profile and on a public employee
profile. **Effect.** The action is produced only when the derived display flag is true: the
reader holds the officer-for-all group, or holds the officer group and is that employee's
approver, or holds the own-records group and the employee is one of the reader's own. In
every other case the operation returns nothing at all rather than refusing, so a person
pressing the button on somebody else's profile simply sees nothing happen.

---

## 9. Company, currency and consistency rules

### AWT-115 — The company of an attendance is the company of its employee

**Scope.** Attendance. **Effect.** There is no company field on the record; every company
condition reads through the employee. An employee with no company is visible from every
company.

### AWT-116 — A Working Schedule, a Resource or a rule set with no company is shared

**Scope.** Working Schedule, Resource, Overtime Ruleset. **Effect.** An empty company means
the record is selectable and readable from every company. A schedule with no company keeps
the full-time reference entered by a person and is never overwritten by a company default
(`AWT-029`).

### AWT-117 — No amount in this domain carries a currency

**Scope.** Every quantity of the domain. **Effect.** Hours are dimensionless decimal
numbers. The only monetary figures in sight are produced by the timesheet comparison
analysis, which multiplies hours by the employee's hourly cost; the cost and its currency
belong to [Human Resources Core](../human-resources-core/) and
[Timesheets](../timesheets/). See
[accounting-effects.md](accounting-effects.md).

### AWT-118 — The settings page writes the two legacy tolerances in one write

**Scope.** The settings page. **Effect.** The two tolerance amounts in favour of the company
and of the employee are not simple mirrors: they are read from the acting company when the
page opens and written back in a single write when the page is saved, and only when at least
one of them actually differs from the stored value. Writing them together avoids recomputing
extra hours several times with a half-saved configuration.

### AWT-119 — Writing a legacy tolerance regenerates that company's extra hours

**Scope.** Company, write of either tolerance amount. **Effect.** The companies whose value
actually changes are collected, and after the write every extra-hours line of every employee
of those companies is deleted and rebuilt.

---

## 10. Rounding and precision rules

### AWT-120 — Hours are decimal, never hours and minutes

**Scope.** Every stored quantity of time. **Effect.** `8.5` is eight hours and thirty
minutes and `7.75` is seven hours and forty-five minutes. Screens render decimal hours in an
hours-and-minutes notation, but no stored value is ever an hours-and-minutes string. The
conversion is specified in
[working-schedule-algorithms.md, chapter 1.3](working-schedule-algorithms.md#13-decimal-hours-and-their-conversion).

### AWT-121 — The rounding points of the domain

| Quantity | Rounding |
|---|---|
| Average hours per day of a schedule | two decimal places, applied once to the unrounded weekly total divided by the working days per week |
| Total hours per week of a schedule | two decimal places |
| A generated extra-hours amount | four decimal places, half away from zero |
| The day total of a duration measurement | the nearest thousandth of a day |
| The hours total of a duration measurement | **not** rounded |
| Hours of the current month and extra hours of the current month | two decimal places |
| Hours reported to the shared terminal and to the menu-bar widget | two decimal places, for display only |
| Expected hours, worked hours and absence hours of the absence ledger | two decimal places |

### AWT-122 — Comparisons of quantities use a fixed precision

**Scope.** Every threshold test. **Effect.** Tolerance comparisons of extra and missing hours
are made at five decimal places; the equality test between the total hours per week and the
full-time reference is made at three decimal places; the zero test that removes an
ineffective technical attendance is made at three decimal places. The comparison rounds the
difference of the two quantities to that many places and yields minus one, zero or plus one,
which is how the tests avoid the noise of binary fractions.

### AWT-123 — The average per day cannot be derived when no day is worked

**Scope.** Working Schedule. **Condition.** The schedule has no countable period at all.
**Effect.** The number of working days per week is zero and the average hours per day is
defined as zero rather than as an error.

### AWT-124 — The work-time rate falls back to one hundred

**Scope.** Working Schedule. **Condition.** The full-time reference is zero. **Effect.** The
work-time rate is exactly one hundred rather than undefined.

### AWT-125 — The day count of a duration measurement rounds to the nearest thousandth

**Scope.** The duration measurement that returns days and hours together. **Effect.** The day
total is rounded to three decimal places; the hour total is returned unrounded. A rebuild
that rounds the day total to a coarser step will disagree with the absence durations of
[Time Off](../time-off/).

### AWT-126 — Display rounding never changes a stored value

**Scope.** Every screen and every payload sent to a device. **Effect.** A quantity rounded
for display is rounded at the moment of rendering; the stored value keeps its full
precision. The clearest case is the second of two adjacent timing bands, whose computed
length is 2.9997 hours and which is shown as three hours.

---

## 11. Date and time-zone rules

### AWT-127 — Every stored instant is on the universal time scale

**Scope.** Every instant of the domain. **Effect.** Instants are stored without a zone and
are understood as universal time. Conversion to a named zone happens only at the moment a
working period, a local-day boundary or a displayed time is computed.

### AWT-128 — The zone of a working period is the schedule's zone, not the resource's

**Scope.** Every interval generation. **Effect.** A resource in one zone following a schedule
defined in another works during the hours of the **schedule's** zone. Where an operation
accepts an explicit zone, that zone overrides both.

### AWT-129 — The effective zone of an employee has a fixed precedence

**Scope.** The stored date of an attendance, the worked-hours computation, the shortfall
date, the technical attendance of the absence job and the automatic check-out. **Effect.**
The zone of the employee's schedule, failing that the employee's own zone, failing that the
zone of the company's default schedule, failing that universal time. For date-sensitive
lookups the schedule is the one of the Employee Version in force on that date. Note the
contrast with the extra-hours generator, which keys days and weeks on the employee's **own**
zone; the difference is exercised by
[acceptance-criteria.md](acceptance-criteria.md#extra-hours-dates-time-zones-and-regeneration).

### AWT-130 — Interval queries must be given zoned bounds

**Scope.** The attendance-interval operation, the exclusion-interval operation and the
closest-working-moment operation. **Condition.** A bound carries no zone. **Message**,
reproduced: "Provided datetimes needs to be timezoned". **Effect.** The call fails. The
hour-count, duration, planning and unusual-day operations accept bounds without a zone and
interpret them as universal time.

### AWT-131 — A local day runs from the first moment to the last representable moment

**Scope.** Every whole-day construction. **Effect.** A local day runs from `00:00:00.000000`
to `23:59:59.999999`. Midnight belongs to the day that starts, never to the day that ends,
which is why a whole day measures 23.999999… hours and not twenty-four, and why a period
ending at hour twenty-four materialises as the last representable moment of its own day.

### AWT-132 — A week runs Monday to Sunday

**Scope.** The weekly period of a quantity rule, the widening of a regeneration scope, and
the weekly key used to group attendances. **Effect.** Monday is the first day and Sunday the
last; a week bucket is keyed by its Sunday. The single exception is the weekly capping of a
flexible resource, which honours the first day of the week of the reader's active language.

### AWT-133 — Planning searches a bounded horizon

**Scope.** Planning forward or backward by hours and by days. **Effect.** The search examines
one hundred consecutive windows of fourteen days, that is 1400 days, and returns no result
rather than looping when the quantity cannot be consumed. A schedule with no working period
at all therefore answers "no result" immediately in practice.

### AWT-134 — A daylight-saving transition changes the length of a working day

**Scope.** Every interval generation in a zone that observes daylight saving. **Effect.**
Working periods are written in wall-clock time and converted per date, so the day on which
the clocks go forward is one hour shorter in elapsed time and the day on which they go back
is one hour longer, while both still report the written number of hours in the schedule's
own terms. The consequences for each algorithm are worked through in
[working-schedule-algorithms.md, chapter 1.5](working-schedule-algorithms.md#15-daylight-saving-transitions).

---

## 12. Mapping of the former rule identifiers

The two source versions of this folder numbered their rules differently: one used a
catalogue with identifiers of the form `AWT-RULE-nnn`, the other stated its rules inside the
validation tables of its entity and calculation files, without identifiers. The table below
maps every former identifier onto the scheme of this file and names, for the version that
had no identifiers, the chapter that carried the same statement. Rows marked "new" state a
rule that neither version numbered.

| New | Former identifier | Where the other version stated it |
|---|---|---|
| AWT-001 | AWT-RULE-001 | Resource, field table and creation defaults |
| AWT-002 | AWT-RULE-002 | Resource, fully flexible resources |
| AWT-003 | AWT-RULE-003 | Resource, field table |
| AWT-004 | AWT-RULE-004 | Resource, field table |
| AWT-005 | AWT-RULE-005 | Resource, interactive changes |
| AWT-006 | AWT-RULE-006 | Resource, deletion |
| AWT-007 | AWT-RULE-007 | Resource Mixin, creation |
| AWT-008 | AWT-RULE-007 | Resource Mixin, creation |
| AWT-009 | AWT-RULE-007 | Resource Mixin, duplication |
| AWT-010 | new | Resource, idempotent writing |
| AWT-011 | new | Resource, creation in bulk |
| AWT-012 | new | Resource, archival |
| AWT-013 | new | Resource form |
| AWT-014 | new | Resource, multi-company behaviour |
| AWT-015 | new | Resource, purpose |
| AWT-016 | new | Resource Mixin, fields contributed |
| AWT-017 | new | Resource Mixin, fields contributed |
| AWT-018 | new | Resource, field table |
| AWT-019 | new | Working Schedule, field table |
| AWT-020 | AWT-RULE-012 | Working Schedule Line, interactive clamping |
| AWT-021 | AWT-RULE-008 | Working Schedule, validations |
| AWT-022 | AWT-RULE-009 | Working Schedule, validations |
| AWT-023 | AWT-RULE-010 | Working Schedule, validations |
| AWT-024 | AWT-RULE-011 | Working Schedule Line, the week-type function |
| AWT-025 | AWT-RULE-013 | Working Schedule Line, computations in detail |
| AWT-026 | AWT-RULE-014 | Working Schedule Line, validations |
| AWT-027 | AWT-RULE-015 | Working Schedule Line, the work-period test |
| AWT-028 | AWT-RULE-016 | Working Schedule, field table |
| AWT-029 | AWT-RULE-017 | Working Schedule, field table |
| AWT-030 | AWT-RULE-018 | Working Schedule, switching between one-week and two-week mode |
| AWT-031 | AWT-RULE-018 | Working Schedule, switching to duration-based entry |
| AWT-032 | new | Working Schedule, multi-company behaviour |
| AWT-033 | new | Working Schedule, lifecycle |
| AWT-034 | new | Working Schedule Line, copying a line |
| AWT-035 | new | Working Schedule Line, computations in detail |
| AWT-036 | new | Working Schedule, the three shapes |
| AWT-037 | new | Working Schedule Line, the work-period test |
| AWT-038 | new | Working Schedule, deletion |
| AWT-039 | new | Working Schedule, defaults on an empty form |
| AWT-040 | new | Attendance, field table |
| AWT-041 | AWT-RULE-019 | Attendance, validations |
| AWT-042 | AWT-RULE-020 | Attendance, validations |
| AWT-043 | AWT-RULE-021 | Attendance, validations |
| AWT-044 | AWT-RULE-022 | Attendance, validations |
| AWT-045 | new | Attendance, validations |
| AWT-046 | AWT-RULE-023 | Attendance, access check on writing |
| AWT-047 | AWT-RULE-024 | Attendance, lifecycle |
| AWT-048 | AWT-RULE-025 | Employee, the change-state operation |
| AWT-049 | AWT-RULE-026 | Attendance, computed fields in detail |
| AWT-050 | AWT-RULE-027 | Attendance, field table |
| AWT-051 | AWT-RULE-028 | Company, the device and location tracking setting |
| AWT-052 | AWT-RULE-029 | Attendance, field table |
| AWT-053 | AWT-RULE-030 | Attendance, lifecycle |
| AWT-054 | new | Working Time Exclusion, validations |
| AWT-055 | new | Working Time Exclusion, validations |
| AWT-056 | AWT-RULE-073 | Working Time Exclusion, field table |
| AWT-057 | AWT-RULE-059 | Working Time Exclusion, purpose |
| AWT-058 | AWT-RULE-058 | Working Time Exclusion, field table |
| AWT-059 | AWT-RULE-060 | the leave-interval algorithm |
| AWT-060 | AWT-RULE-047 | Attendance, recomputing extra hours |
| AWT-061 | AWT-RULE-061 | Attendance, lifecycle; Working Time Exclusion, effect on attendance overtime |
| AWT-062 | AWT-RULE-048 | Attendance, the recomputation window |
| AWT-063 | AWT-RULE-049 | Attendance, the recomputation window |
| AWT-064 | AWT-RULE-031 | the extra-hours generation algorithm |
| AWT-065 | AWT-RULE-032 | the extra-hours generation algorithm |
| AWT-066 | AWT-RULE-033 | Overtime Rule, validations |
| AWT-067 | AWT-RULE-033 | Overtime Rule, validations |
| AWT-068 | AWT-RULE-034 | Overtime Rule, validations |
| AWT-069 | AWT-RULE-035 | Overtime Rule, field table |
| AWT-070 | AWT-RULE-036 | timing rules |
| AWT-071 | AWT-RULE-037 | quantity rules |
| AWT-072 | AWT-RULE-038 | quantity rules |
| AWT-073 | AWT-RULE-039 | quantity rules |
| AWT-074 | AWT-RULE-040 | the two tolerances |
| AWT-075 | AWT-RULE-041 | the two tolerances |
| AWT-076 | AWT-RULE-042 | undertime |
| AWT-077 | AWT-RULE-043 | undertime |
| AWT-078 | AWT-RULE-044 | the top-level generation |
| AWT-079 | AWT-RULE-045 | Attendance Overtime Line, field table |
| AWT-080 | AWT-RULE-046 | the top-level generation |
| AWT-081 | new | Attendance Overtime Line, constraints |
| AWT-082 | new | Attendance Overtime Line, constraints |
| AWT-083 | AWT-RULE-050 | combining pay rates |
| AWT-084 | new | Attendance Overtime Line, field table |
| AWT-085 | new | manual adjustment of an amount |
| AWT-086 | new | quantity rules, step one |
| AWT-087 | new | undertime |
| AWT-088 | new | Attendance, field table |
| AWT-089 | new | Attendance Overtime Line, approval operations |
| AWT-090 | AWT-RULE-051 | the automatic check-out arithmetic |
| AWT-091 | AWT-RULE-052 | the automatic check-out arithmetic |
| AWT-092 | AWT-RULE-053 | the automatic check-out arithmetic |
| AWT-093 | new | the automatic check-out arithmetic, selection |
| AWT-094 | AWT-RULE-054 | the absence-detection arithmetic |
| AWT-095 | AWT-RULE-055 | the absence-detection arithmetic |
| AWT-096 | AWT-RULE-056 | the absence-detection arithmetic |
| AWT-097 | AWT-RULE-056 | the absence-detection arithmetic |
| AWT-098 | AWT-RULE-057 | the two unattended jobs |
| AWT-099 | new | the two unattended jobs |
| AWT-100 | AWT-RULE-062 | Attendance, multi-company behaviour |
| AWT-101 | AWT-RULE-063 | Attendance, company scoping |
| AWT-102 | AWT-RULE-063 | Attendance Overtime Line, multi-company behaviour |
| AWT-103 | AWT-RULE-064 | Attendance, field table |
| AWT-104 | AWT-RULE-065 | Employee, behaviours overridden |
| AWT-105 | AWT-RULE-065 | User, the clean-up operation |
| AWT-106 | AWT-RULE-066 | Overtime Ruleset, multi-company behaviour |
| AWT-107 | AWT-RULE-066 | Employee Version, field table |
| AWT-108 | AWT-RULE-067 | Working Schedule, access |
| AWT-109 | AWT-RULE-068 | Working Time Exclusion, access |
| AWT-110 | AWT-RULE-069 | the shared-terminal endpoints |
| AWT-111 | AWT-RULE-069 | the shared-terminal endpoints |
| AWT-112 | AWT-RULE-070 | Company, behaviours added |
| AWT-113 | AWT-RULE-071 | Attendance, field table |
| AWT-114 | new | Public Employee, the monthly-hours action |
| AWT-115 | AWT-RULE-072 | Attendance, company scoping |
| AWT-116 | AWT-RULE-074 | Working Schedule, multi-company behaviour |
| AWT-117 | AWT-RULE-075 | notation and precision |
| AWT-118 | new | Configuration Settings |
| AWT-119 | new | Company, behaviours added |
| AWT-120 | AWT-RULE-076 | conventions, decimal hours |
| AWT-121 | AWT-RULE-077 | the several rounding chapters |
| AWT-122 | AWT-RULE-078 | notation and precision |
| AWT-123 | AWT-RULE-079 | average hours per day |
| AWT-124 | AWT-RULE-080 | the full-time reference and the work-time rate |
| AWT-125 | new | counting days |
| AWT-126 | new | timing rules, worked example I |
| AWT-127 | AWT-RULE-081 | two kinds of moment |
| AWT-128 | AWT-RULE-082 | which zone wins |
| AWT-129 | AWT-RULE-083 | Attendance, computed fields in detail |
| AWT-130 | AWT-RULE-084 | the nearest working moment |
| AWT-131 | AWT-RULE-085 | notation and precision |
| AWT-132 | AWT-RULE-086 | day-of-week numbering, week start and the week key |
| AWT-133 | AWT-RULE-087 | planning by hours and by days |
| AWT-134 | new | daylight saving transitions |

---

## 13. Reconciliation notes

Both source versions were merged rule by rule. Every rule of the numbered catalogue survives
in the table above, and every validation stated in the other version's entity and
calculation files has been given a number. The following points differed and were resolved
against the source.

1. **What the officer record rule actually selects.** One version described it twice and
   differently: once as "the employee names the reader as attendance approver", once as
   "the employee names the reader as approver, **or** the employee is the reader". The
   condition has two branches but both require the approver to be the reader; being the
   employee is not sufficient. The first wording is correct and is what `AWT-101` states.
2. **The number of overlap validations.** One version listed three separate rules with three
   messages. There are two validations — one on the ordering of the two instants, one on the
   three overlap cases — and they raise four messages between them. The rules are kept
   separate for citation, and `AWT-045` records that they run over a whole batch.
3. **Where the "regeneration forces a review" rule belongs.** One version placed it among
   the approval rules, the other among the recomputation steps. It governs the generator, so
   it opens [chapter 6](#6-extra-hours-rules) as `AWT-060`, which is the identifier the
   entity and calculation files already cite.
4. **Automatic check-out and fully flexible employees.** One version asserted that such an
   employee is never closed automatically. The selection filter excludes employees whose
   *schedule* is flexible, and a fully flexible employee has no schedule to test. The
   observed behaviour and the corrected behaviour are both recorded in `AWT-093` as a
   **compatibility finding**.
5. **The two legacy tolerance settings.** One version called them hidden, the other listed
   them among the settings. Both are right in part: they exist on the settings page but
   their block is permanently invisible, and they are written in one write. That is
   `AWT-118`, and their effect on the stored data is `AWT-119`.
6. **The employee tolerance of a timing rule.** One version implied that both tolerances
   apply to both families. Only the employer tolerance applies to a timing rule, because a
   timing rule can never produce a shortfall; `AWT-074` and `AWT-075` say so explicitly.
7. **The day-count rounding.** One version wrote "the closest sixteenth of a day", following
   a stale comment in the source. The arithmetic rounds to the nearest **thousandth** of a
   day; that is `AWT-125`.
8. **Rules that neither version numbered** — the idempotent write, the batch-failure
   behaviour, the exclusion ordering and overlap validations, the two database checks on an
   extra-hours line, the legitimacy of overlapping lines, the status computation running
   only while empty, the elevated rights of the two jobs, the silent monthly-hours action
   and the daylight-saving consequence — were extracted from the entity and calculation
   files of both versions and from the source, and are marked "new" in the mapping table.
