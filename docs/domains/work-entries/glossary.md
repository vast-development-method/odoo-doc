# Work Entries — Glossary

Every term this folder uses in a precise sense, defined. Terms are listed alphabetically. Where a
term names a stored value, the stored value is reproduced in code font; where it names a screen
label or a message, the text is reproduced in quotation marks. Terms owned by another domain are
defined here only to the depth this folder needs, with a link to the folder that owns them.

---

## A

**Absence entry.** A Work Entry whose Work Entry Type carries the absence flag. An absence entry
means *the employee did not work these hours, and the reason is recorded*. Its duration is not the
clock span of the interval it came from: it is the theoretical duration the working schedule
prescribed for that span. See [calculations.md, chapter 8](calculations.md#8-the-post-processing-pass-intervals-to-day-rows).

**Absence companion.** The capability package that bridges the Work Entries domain to the
[Time Off](../time-off/README.md) domain. It adds the absence link on a Work Entry, the work entry
kind on a Time Off Type, the generation of absence entries when a request is validated, and the
regeneration when a request is refused or cancelled. Its interface name is "Time Off in Payslips".

**Absence flag.** The true/false field `is_leave` ("Time Off") on a Work Entry Type. True means the
kind describes time not worked. The flag drives five distinct behaviours: how the duration of a
generated interval is measured, whether an entry outside the working schedule becomes a conflict,
whether a Working Schedule Line contributes to the schedule's global attendances and therefore to
its weekly hours, whether a schedule line counts as a work period, and which post-processing branch
the interval takes. See [entities.md, section 3.6](entities.md#36-the-absence-flag-and-its-two-mirrors).

**Absence request.** A Time Off Request, owned by the [Time Off](../time-off/README.md) domain. This
folder consumes it in exactly one state, the validated state, and reacts to three departures from
it: refusal, reversion to an earlier approval step, and cancellation by the requester.

**Acting company.** The company under whose authority an operation runs. Generation deliberately
switches the acting company for each group of versions it processes, so that the Working Time
Exclusions it reads belong to that company and are never mixed across companies. See
[calculations.md, chapter 3](calculations.md#3-entry-points-and-the-company-and-time-zone-grouping).

**Attendance interval.** An interval produced by expanding a Working Schedule over a window: a start
instant, a stop instant and the Working Schedule Line that produced it. It is the raw material of
generation. The expansion itself belongs to
[Attendances and Working Time](../attendances-and-working-time/working-schedule-algorithms.md).

---

## B

**Bypassing payroll code.** A payroll code listed by the extension point that lets a country package
declare "an absence of this kind outranks a global closure". When an interval is covered both by a
company closure and by a validated absence whose kind carries a bypassing code, the absence wins and
the closure is ignored. The list is empty in the specified system; only country packages fill it.
See [calculations.md, section 7.3](calculations.md#73-the-precedence-ladder-for-an-absence-interval).

---

## C

**Calendar filter.** A Work Entry Employee Filter record: one row per pair of a login user and an
employee, recording that the user has pinned that employee into the work entry calendar and whether
the employee is currently ticked. See [entities.md, chapter 5](entities.md#5-work-entry-employee-filter-hruserworkentryemployee-table-hr_user_work_entry_employee).

**Company closure.** A Working Time Exclusion with no resource: a span during which nobody in the
company works. A public holiday is the usual example. Compare **individual exclusion**.

**Conflict.** The state `conflict`, labelled "In Conflict". An entry is in conflict when at least one
of four conditions holds on it or on its day. A day carrying a conflicting entry cannot be validated.
The four conditions are enumerated in
[business-rules.md, chapter 5](business-rules.md#5-the-four-conflict-conditions).

**Conflict re-check window.** The date range over which conflicts are recomputed around a change.
For a change made through a Work Entry it is the range spanned by the entries being changed. For a
change made through an absence request it is widened by one whole day at each end, because the
instants of a request are derived from its requested dates in the employee's time zone and may fall
on the neighbouring calendar day. See
[business-rules.md, chapter 6](business-rules.md#6-the-conflict-re-check-window).

**Contract period.** The span between the contract start date and the contract end date of an
Employee Version. Generation ignores a version with no contract start date entirely. Owned by
[Human Resources Core](../human-resources-core/calculations.md#5-contract-periods).

---

## D

**Day book.** The informal name for the set of Work Entries of one employee: one line per calendar
date and kind. This folder uses it as a collective noun; no record carries that name.

**Display code.** The optional field `display_code` on a Work Entry Type: at most three characters,
shown as a badge on a calendar cell. Purely cosmetic. Nothing in the platform keys off it.

**Duration.** The field `duration` on a Work Entry: a number of hours expressed as a decimal, where
seven and a half hours is written as seven point five. A work entry has a duration and a date and no
clock times at all.

---

## E

**Effective end date.** The last calendar date on which an Employee Version is in force: the earlier
of the day before the next version's version date and the contract end date, or the contract end
date when there is no next version. Owned by
[Human Resources Core](../human-resources-core/calculations.md#4-effective-start-and-end-dates-of-a-version).

**Effective start date.** The first calendar date on which an Employee Version is in force: the later
of its version date and its contract start date, or its version date when there is no contract start
date. Owned by Human Resources Core.

**Exclusion.** Short for Working Time Exclusion: a span carved out of a Working Schedule during which
work does not happen. Owned by
[Attendances and Working Time](../attendances-and-working-time/entities.md#5-working-time-exclusion).
Two kinds matter here: a **company closure**, which has no resource and applies to everybody on a
schedule, and an **individual exclusion**, which names a resource and usually carries a link back to
the absence request that created it.

**External code.** The optional field `external_code` on a Work Entry Type, carried only so that an
export can use a third party's own vocabulary. Nothing inside the platform reads it.

**Extra hours flag.** The true/false field `is_extra_hours` on a Work Entry Type, labelled "Added to
Monthly Pay". True means the hours are a bonus on top of the basic salary rather than part of it. A
payroll capability reads it; this domain only stores it.

---

## F

**Flexible-hours schedule.** A Working Schedule marked as flexible: the employee owes a number of
hours per day and per week but not at fixed clock times. Generation treats an absence on such a
schedule differently depending on whether it falls inside one calendar day or spans several. See
[calculations.md, section 5.3](calculations.md#532-branch-b-a-flexible-hours-schedule).

**Forced regeneration.** A generation run with the force flag set. It ignores the generated-through
markers, nullifies every non-validated entry already present in the range, and writes the range
again from scratch. The only way to make the platform rebuild a range it believes it has already
covered.

**Fully flexible schedule.** An Employee Version with no working schedule at all. Generation produces
one entry per calendar day covering the whole requested window for such a version, so a full day
yields twenty-four hours. See
[calculations.md, section 4.3](calculations.md#43-a-version-with-no-working-schedule).

---

## G

**Generated-from marker.** The instant field `date_generated_from` ("Generated From") on an Employee
Version: the earliest instant for which the day book of that version is believed complete.

**Generated-to marker.** The instant field `date_generated_to` ("Generated To") on an Employee
Version: the latest instant for which the day book of that version is believed complete.

**Generated window.** The span between the two markers. Generation writes only outside it, unless it
is forced.

**Generation.** The act of deriving Work Entries from a Working Schedule, the exclusions that overlay
it and the validated absences of the employee, for a period and a set of versions. Specified end to
end in [calculations.md, chapters 3 to 12](calculations.md#3-entry-points-and-the-company-and-time-zone-grouping).

**Generation source.** The selection `work_entry_source` on an Employee Version, required, default
`calendar` "Working Schedule". It names where the day book comes from. `calendar` is the only value
the specified system offers; see
[business-rules.md, chapter 12](business-rules.md#12-the-generation-source-extension-point).

**Global attendance.** A Working Schedule Line that applies to every resource on the schedule, as
opposed to one restricted to a single resource. This domain narrows the set: a line whose work entry
kind carries the absence flag is excluded from it, and therefore from the schedule's hours-per-week
figure.

---

## H

**Half day.** The unit in which a Working Schedule Line contributes to a day count. A morning or
afternoon line counts as half a day when its span is at most three quarters of the schedule's
hours-per-day figure, and as a whole day otherwise. A full-day line always counts as one day; a break
line counts as zero. See [calculations.md, chapter 11](calculations.md#11-the-half-day-rounding-rule-and-the-day-count).

---

## I

**Individual exclusion.** A Working Time Exclusion that names a resource. The exclusion a validated
absence request creates is of this kind and carries a link back to the request.

**Interval.** A triple of a start instant, a stop instant and a payload record. The generation engine
manipulates four families of intervals — attendance intervals, exclusion intervals, absence intervals
and worked-absence intervals — using the union, intersection and difference operations specified in
[Attendances and Working Time](../attendances-and-working-time/working-schedule-algorithms.md).

**In payslip.** The label of the state `validated`. It means the entry is locked: a payroll run has
taken it, it may not be edited, deleted, regenerated or swallowed by an absence.

---

## L

**Last generation date.** The calendar date field `last_generation_date` ("Last Generation Date") on
an Employee Version. It records the date of the last generation *attempt*, written before any entry
is produced, and is read by the daily scheduled job so that one version is not revisited twice on the
same day.

---

## N

**Nullifying values.** The set of field values a forced regeneration writes onto the entries it
supersedes. In the specified system the set contains exactly one field, the archived flag `active`,
set to false — which, through the coupling described in
[entities.md, section 4.6](entities.md#46-the-coupling-between-the-state-and-the-archived-flag), also
writes the state `cancelled`. The set is an extension point: a country package may add its own
fields to it.

---

## O

**Out of contract.** The shipped Work Entry Type with payroll code `OUT`, display code `OoC`. It
labels time inside a payroll period during which the employee had no contract. This domain ships the
type; only a payroll capability produces entries of it.

---

## P

**Payroll code.** The required field `code` ("Payroll Code") on a Work Entry Type. It is the
identifier every downstream payroll rule and every statutory export keys off, and it is unique within
a country scope. Its help text warns: "Careful, the Code is used in many references, changing it
could lead to unwanted changes."

**Pay rate.** The field `amount_rate` ("Rate") on a Work Entry Type and, copied at creation time, on a
Work Entry. It is a multiplier: one is ordinary pay, one and a half is time and a half, two is double
time, zero point seven five is seventy-five percent of pay. Displayed as a percentage.

**Period.** In this folder, always a closed range of calendar dates, inclusive at both ends. Compare
**window**.

**Post-processing pass.** The stage of generation that converts intervals into day rows: it splits a
span that crosses local midnight, assigns each row its calendar date in the schedule's time zone,
computes each row's duration, drops rows of zero duration and merges rows sharing a date, a kind, an
employee, a version and a company. Specified in
[calculations.md, chapter 8](calculations.md#8-the-post-processing-pass-intervals-to-day-rows).

---

## Q

**Quick replace.** The calendar operation that replaces the entries of every selected day with one
entry of a chosen kind, keeping each day's existing total hours where there are any and regenerating
the day where there are none. See
[interfaces.md, section 5.3](interfaces.md#53-the-multiple-selection-buttons).

---

## R

**Regeneration.** A forced generation of a range, run through the Work Entry Regeneration Wizard. See
**forced regeneration**.

**Regeneration wizard.** The transient entity Work Entry Regeneration Wizard, labelled "Regenerate
Employee Work Entries". It collects a set of employees and a range, clamps the range to what has
actually been generated, excludes employees holding a validated entry in the range, and runs a forced
generation.

---

## S

**Sentinel state of the markers.** The condition in which the generated-from marker and the
generated-to marker of a version are equal. It means *nothing has ever been generated for this
version*. Generation detects it and moves both markers to the start of the requested window before
doing anything else, so that a version created long ago does not cause years of history to be
generated. See [calculations.md, section 9.1](calculations.md#91-the-sentinel-reset).

**Static generation.** The property of a version whose generation source is `calendar`: the same day
book would be produced every month from the same schedule. Static versions take a cheaper path
through the engine and are processed first by the scheduled job.

---

## T

**Theoretical duration.** The number of hours the working schedule prescribes for a span, obtained by
re-expanding the schedule over that span and summing the resulting attendance intervals, ignoring
exclusions. Absence entries are measured this way rather than by their own clock span.

**Time zone of generation.** The time zone in which the requested dates are turned into instants and
in which each produced interval is dated. It is the schedule's time zone, falling back to the
employee's time zone when the version has no schedule, and to the universal time scale when neither
has one. It is the single most consequential parameter of generation, because it decides which
calendar day an interval lands on and therefore how many rows are produced.

---

## U

**Unusual day.** A calendar day on which the employee is not expected to work, which the calendar
greys out. The set is derived from the acting company's default working schedule. See
[interfaces.md, section 4.4](interfaces.md#44-unusual-days).

---

## V

**Validation, of a work entry.** The operation that moves entries from `draft` to `validated`. It
first runs the four conflict conditions over the selection; if any of them marks anything, nothing is
validated and the operation reports failure. Not to be confused with the validation of an absence
request, which belongs to [Time Off](../time-off/README.md).

**Version.** Short for Employee Version, the dated record of an employee's employment terms, owned by
[Human Resources Core](../human-resources-core/entities.md#3-employee-version-hrversion-table-hr_version).
It names the working schedule, carries the contract period, and hosts the generation markers and the
whole generation engine.

---

## W

**Window.** In this folder, always a range of instants, as opposed to a **period**, which is a range
of calendar dates. Generation is handed a period and converts it into a window; the conversion point
is stated exactly in [calculations.md, chapter 3](calculations.md#3-entry-points-and-the-company-and-time-zone-grouping)
because it is a frequent source of off-by-one behaviour.

**Work entry.** One line of the day book: one employee, one calendar date, one kind, one duration in
hours, one employment version, one company, one state.

**Work entry kind.** Everyday wording for a Work Entry Type.

**Worked absence.** An interval covered by a Working Time Exclusion whose time type is the
non-absence value — training, for instance. The employee is away from the ordinary schedule but the
time counts as worked. Worked absences are separated from ordinary absences early in generation,
carry the kind of the exclusion that produced them, and are measured by the theoretical schedule only
when their kind carries the absence flag.

**Working schedule.** The weekly pattern of working hours, owned by
[Attendances and Working Time](../attendances-and-working-time/entities.md#3-working-schedule). Named
by an Employee Version, it is the source from which the day book is expanded.

**Working schedule line.** One line of that weekly pattern: a weekday, a day period, a start hour and
an end hour. This domain adds a work entry kind to it, defaulted to the shipped ordinary-attendance
kind.
