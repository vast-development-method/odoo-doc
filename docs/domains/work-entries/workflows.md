# Work Entries — Workflows

The operational procedures of the domain, step by step. Each procedure names the role that performs
it, the records each step creates or changes, the operations it invokes and the conditions under which
it fails. Rules are cited by their identifier from
[business-rules.md](business-rules.md#14-the-table-of-rule-identifiers); algorithms by their chapter
in [calculations.md](calculations.md).

---

## 1. The roles involved

| Role | What it may do here |
|---|---|
| Human Resources Officer | Read and write the catalogue of kinds only for reading; read, create and write work entries; never delete one; manage their own calendar filter |
| Human Resources Manager | Everything an officer may do, plus create, change and archive work entry kinds, set the generation source on a version, and run the regeneration wizard |
| Settings group | In addition, delete a work entry — subject to `WKE-035`, which still protects validated entries |
| The platform's own scheduled runner | Runs the daily generation job as the root user |
| Payroll capability | Reads the day book and calls the validation operation; not part of this domain |
| Employee | Has no access to work entries at all; an employee sees only the absence requests that produce them |

---

## 2. Setting up the catalogue of work entry kinds

Performed once, by a human resources manager, before any payroll period is run.

1. Review the shipped catalogue. The universal kinds — ordinary attendance, overtime, out of contract,
   generic time off, compensatory time off, home working, unpaid, sick time off and paid time off —
   are present in every installation. The country-specific kinds of the installed countries are
   present as well. The whole shipped catalogue is enumerated in
   [configuration.md, chapter 3](configuration.md#3-the-shipped-catalogue-of-work-entry-kinds).
2. For each kind the payroll of this company needs and which is not shipped, create one: give it a
   name, a payroll code, optionally a display code of at most three characters, a colour, a pay rate,
   the absence flag if the time is not worked, and the extra-hours flag if the hours are a bonus on
   top of the basic salary.
   - *Fails when* the payroll code collides with an existing kind in the same country scope
     (`WKE-006`), or the name or the payroll code is missing (`WKE-001`, `WKE-002`).
3. Set the country on the kinds that belong to a single country's payroll vocabulary, and leave it
   empty on the kinds every company may use.
   - *Fails when* the kind is the shipped ordinary-attendance kind (`WKE-004`) or any work entry
     already references the kind (`WKE-005`).
4. Attach the kinds to the places that produce them:
   - on each Working Schedule Line, the kind the line produces — defaulted to the shipped
     ordinary-attendance kind;
   - on each Working Time Exclusion that represents a public holiday or a company closure, the kind
     that closure produces;
   - on each Time Off Type, the kind an absence of that kind produces.
5. Archive the kinds this company will not use. Archiving hides them from every list and changes
   nothing else (`WKE-009`).

**Records changed:** Work Entry Type; Working Schedule Line; Working Time Exclusion; Time Off Type.

---

## 3. Preparing an employee for generation

Performed by a human resources officer when an employee is hired, and again whenever the employment
terms change.

1. Create or open the employee's Employee Version for the period.
2. Ensure the version carries a **contract start date**. A version with none is skipped by generation
   altogether — silently, with no message. This is the single most common reason for an empty day
   book.
3. Ensure the version names a **working schedule**, unless the employee is deliberately fully
   flexible. A version whose generation source is the working schedule but which names no schedule
   raises the invalid-source indicator and the warning "Invalid option: For fully flexible calendars,
   the work entry source cannot be 'Working Hours'." (`WKE-061`), and generates one whole-day interval
   per calendar day.
4. Ensure the working schedule names a **time zone**, or the employee does, or the company's schedule
   does; otherwise generation fails with "Missing timezone for work entries generation." (`WKE-063`).
5. Leave the generation source at `calendar` "Working Schedule" unless a package has added another
   source.
6. Note the two markers. On a freshly created version both read today at midnight, which is the
   sentinel meaning nothing has been generated.

**Records changed:** Employee Version.

---

## 4. Generating a month automatically

Performed by the platform, once a day, with no human involvement.

1. The scheduled job computes its period: the first day of the current month to the last day of the
   next month, as specified in [calculations.md, chapter 16](calculations.md#16-the-window-arithmetic-of-the-daily-job).
2. It selects every version, of every employee, active or archived, that overlaps that period by at
   least one contracted day.
3. It narrows the selection to versions that still have something to do: those whose generated-from
   marker is later than the period start **or** whose generated-to marker is earlier than the period
   stop, and whose last generation date is empty or earlier than today.
4. If nothing remains, the job ends.
5. It counts what remains, then narrows the selection to the versions of the **first** remaining
   version's company. Versions of other companies wait for a later run; mixing companies would mix
   their Working Time Exclusions.
6. It sorts the remaining versions so that statically generated ones come first, and takes the first
   hundred.
7. It runs generation over the period, not forced, with the language of the job's own user.
8. If more than a hundred versions had something to do, it re-triggers itself so that the next batch
   runs immediately.

**Records changed:** Work Entry (created); Employee Version (the two markers and the last generation
date).

*Fails when:* a version has no time zone anywhere (`WKE-063`), which aborts the whole batch. The
job's next run retries.

---

## 5. Generating on demand from the calendar

Performed by a human resources officer, implicitly, simply by looking at the calendar.

1. The officer opens the work entries of one employee, from the employee form's "Work Entries" button
   or from a saved navigation path.
2. Every time the calendar's range changes — a new month, a new filter — the client calls the
   employee-level generation entry point for the employee of the view and the visible range, not
   forced.
3. Generation runs as specified in [calculations.md, chapter 6](calculations.md#6-the-generation-algorithm-step-by-step).
   Only the part of the range outside the markers is produced.
4. The client then reloads the events and, separately, reads the officer's six most recently used
   kinds over the last three months so that the multiple-selection toolbar can offer them as shortcut
   buttons.

**Records changed:** Work Entry (created); Employee Version (markers, last generation date).

*Fails silently when:* the version has no contract start date; the employee has no version at all; the
period lies entirely outside every version's effective window. In each case the calendar simply shows
nothing for the days concerned.

---

## 6. Regenerating a range by hand

Performed by a human resources manager when the day book of a period no longer reflects reality —
after a schedule was corrected retroactively, or after somebody edited entries by hand and wants the
derived ones back.

1. Open the regeneration wizard, either from the calendar's "Reset" button, which pre-fills the
   employees of the current view and the visible range, or from its own window action.
2. Choose the employees. The list offers employees of the acting companies that have at least one
   version.
3. Choose the period. The to-date defaults to the last day of the month containing the from-date, by
   the arithmetic of [calculations.md, section 15.2](calculations.md#152-the-default-end-of-the-range).
4. As the choices change, the form clamps them, as specified in
   [calculations.md, section 15.3](calculations.md#153-the-interactive-clamping): it swaps an inverted
   pair, pulls the from-date up to the earliest available date with the message "The earliest
   available date is " followed by that date, and pulls the to-date down to the latest available date
   with the message "The latest available date is " followed by that date.
5. The form colours in red every selected employee that holds a validated entry inside the range and
   warns: "Employees in red will be skipped because they have at least one validated work entry."
6. The form warns, always: "Warning: The work entry regeneration will delete all manual changes on the
   selected period."
7. If at least one selected employee is regenerable, the confirming button is offered; otherwise a
   disabled button is shown in its place.
8. On confirmation the operation runs:
   a. `WKE-047`, `WKE-048` and `WKE-049` are checked, in that order, each refusing with its own
      message.
   b. The employee set is narrowed by `WKE-050`.
   c. The range is clamped again by [calculations.md, section 15.4](calculations.md#154-the-clamping-applied-at-run-time).
   d. If the narrowed set is empty, the operation returns having done nothing.
   e. Otherwise the employee-level generation entry point is called with the force flag set.
9. Forced generation nullifies every non-validated entry of each version in the clamped range —
   archiving them and thereby cancelling them (`WKE-053`) — and writes the range again from the
   schedule, the exclusions and the validated absences.

**Records changed:** Work Entry (the superseded ones archived and cancelled; new ones created);
Employee Version (the markers, only outward and only from what was produced).

---

## 7. Regeneration triggered by a version change

Performed by the platform, automatically, when a human resources officer changes an employment term
that changes the produced day book.

1. The officer writes a new **working schedule** or a new **generation source** onto an Employee
   Version.
2. If the same write also changed the contract start date, the contract end date or the version date,
   the out-of-period removal of [calculations.md, section 10.3](calculations.md#103-removal-outside-the-contract-period)
   runs first, deleting entries outside the new period and pulling the corresponding marker back.
3. The platform then computes the overlap of the version's effective window and its generated window:

   ```formula
   recompute from = the later of  ( the version's effective start date ,
                                    the date part of the generated-from marker )
   recompute to   = the earlier of( the version's effective end date, or the largest representable
                                    date when there is none ,
                                    the date part of the generated-to marker )
   ```

4. If the two are equal, or the version has no employee, nothing happens.
5. Otherwise a regeneration wizard is built for that version's employee over that range and run with
   the skip-validation marker and with archived records included (`WKE-051`). The three guards do not
   fire; `WKE-050` does, so an employee holding a validated entry in the range is skipped and nothing
   at all is regenerated for them.
6. The whole behaviour is suppressed when the write happens inside a salary simulation: a simulation
   must never touch the real day book.

**Records changed:** Work Entry; Employee Version.

**Worked outcome.** An employee on a forty-hour schedule has January generated: twenty-three
attendance rows totalling one hundred and eighty-four hours. A second version is created from the
tenth of January naming a thirty-five-hour schedule. Generation is asked for January again. The day
book becomes twenty-three rows totalling one hundred and sixty-eight hours: seven days at eight hours
under the first version and sixteen days at seven hours under the second.

---

## 8. Validating work entries

Performed by a payroll capability, or by an automation, at the close of a payroll period. No interface
of this domain offers the operation; the domain supplies it and the payroll domain calls it.

1. The caller selects the entries of the period — typically every entry of one employee between two
   dates.
2. The caller invokes the validation operation.
3. Entries already validated are removed from the selection (`WKE-031`).
4. The four conflict conditions run over the remainder (`WKE-020` to `WKE-023`).
5. If any condition marked anything, the operation reports failure and writes no state. The entries it
   marked stay in conflict; a human must resolve them.
6. If nothing was marked, every entry of the narrowed selection moves to `validated` in one write.
7. From that moment the entries are locked: not deletable (`WKE-035`), not regenerable (`WKE-033`),
   not swallowable by an absence (`WKE-034`), not editable through the interface (`WKE-032`), and
   their absence requests are no longer cancellable by their requester (`WKE-045`).

**Records changed:** Work Entry.

---

## 9. Correcting a conflict

Performed by a human resources officer.

1. Open the work entries with the conflict filter applied, or open the conflict window action, which
   applies that filter by default.
2. For each conflicting day, read the form's explanation: either "This work entry cannot be validated.
   The work entry type is undefined." or "The amount of work on the day should not exceed 24 hours."
3. Apply the correction the condition calls for:

| Condition | Correction |
|---|---|
| `WKE-020`, no kind | Set a kind on the entry |
| `WKE-021`, the day's total leaves the range | Reduce a duration, delete a manual entry, or archive one; the day is re-examined on the write |
| `WKE-022`, an absence outside the schedule | Either correct the absence request so that it falls on days the employee works, or correct the working schedule, or archive the entry |
| `WKE-023`, the day already holds a validated entry | Archive the surplus entry; the validated one is authoritative |

4. Every one of those corrections writes one of the five fields that trigger the check (`WKE-024`), so
   the reset pass runs, the whole window returns to `draft`, and the four conditions run again. A day
   that no longer qualifies comes out of conflict by itself; no explicit clearing operation exists.

**Records changed:** Work Entry; possibly Time Off Request; possibly Working Schedule.

---

## 10. Splitting a work entry

Performed by a human resources officer from the calendar, when a day recorded as one kind should be
two.

1. Click the event in the calendar and open its popover. The split action is offered only when the
   entry's duration is at least one hour and the entry is not validated.
2. A dialogue opens, pre-filled with half the entry's duration, the entry's description, its kind, its
   employee and its date. The employee, the date, the company and the state are read-only in this
   dialogue.
3. Change the duration and the kind of the part being split off, and confirm.
4. The split operation runs (`WKE-065`):
   a. Refuse with "You can't split a work entry with less than 1 hour." when the entry is shorter than
      an hour.
   b. Refuse with "Split work entry duration has to be less than the existing work entry duration."
      when the requested part is not strictly smaller than the whole.
   c. Reduce the original entry's duration by the requested amount.
   d. Copy the original entry. The copy starts in `draft`, because the state is not copied; it carries
      the same employee, version, date, company, pay rate and absence link.
   e. Write the requested duration, kind and description onto the copy.
5. The calendar reloads.

**Records changed:** Work Entry (one changed, one created).

**Worked outcome.** An eight-hour ordinary attendance row on 3 March is split by three hours into
overtime. Afterwards the day carries a five-hour ordinary attendance row and a three-hour overtime
row; the day total is still eight hours and no conflict arises.

---

## 11. An absence is validated

Performed by whoever approves absences, in the [Time Off](../time-off/README.md) domain. This domain
reacts.

1. The absence request reaches its validated state. The Time Off domain creates the Working Time
   Exclusion that represents it, and this domain's companion adds the work entry kind of the request's
   absence kind onto that exclusion (`WKE-041`).
2. This domain's companion then runs, with elevated rights:
   a. For each request, find the versions of its employee that overlap the request's dates by at least
      one contracted day.
   b. For each such version, produce values **only if** the request's stop is not earlier than that
      version's generated-from marker and the request's start is not later than its generated-to
      marker (`WKE-042`). An absence beyond the generated window produces nothing now; its rows appear
      when generation next reaches that period.
   c. Compute the values over the request's whole days, pass them through the post-processing pass and
      create the resulting entries. They carry the absence kind's work entry kind and the absence link.
   d. Read every entry of the employee in the range spanned by the batch, split them into the rows
      just created and the rows that existed before, and compare them as whole-day intervals.
   e. Every pre-existing, non-validated entry whose day lies at least partly outside the new rows
      loses its absence link.
   f. Every other pre-existing, non-validated entry is archived, and thereby cancelled (`WKE-043`).
3. A separate effect fires on the create and on every write of the request itself: the conflict
   re-check window is opened, one day wider at each end (`WKE-026`), and every non-validated,
   non-cancelled entry of the affected employees inside it is reset out of conflict and re-examined.

**Records changed:** Working Time Exclusion (created by the Time Off domain, with a kind added here);
Work Entry (created, archived, or unlinked).

**Worked outcome, a full day.** A five-day employee has Monday to Friday generated at eight hours
each. A one-day absence is validated for the Wednesday. Afterwards Wednesday carries one absence row
of eight hours of the absence's kind, linked to the request, and the former eight-hour attendance row
of that Wednesday is archived and cancelled. Monday, Tuesday, Thursday and Friday are untouched. The
week still totals forty hours.

**Worked outcome, two hours.** The same employee takes a two-hour absence on the Wednesday, from 10:00
to 12:00. Afterwards Wednesday carries a six-hour attendance row and a two-hour absence row. The
former eight-hour row is archived. The week still totals forty hours.

---

## 12. An absence is refused or cancelled

Performed by an approver, an administrator or the requester, in the Time Off domain. Three distinct
operations lead here and all three do the same thing (`WKE-044`): refusing a request, moving a
validated request back to an earlier approval step, and the requester cancelling their own request.

1. The Time Off domain performs its own part of the operation and removes the Working Time Exclusion.
2. This domain's companion finds every work entry whose absence link points at one of the requests,
   with elevated rights.
3. It archives them all, which also cancels them.
4. For each archived entry, it recomputes the values of that entry's version over that entry's whole
   date, from midnight to the last representable microsecond, passes them through the post-processing
   pass, and creates the result. The ordinary attendance of those days is restored.

**Records changed:** Working Time Exclusion (removed); Work Entry (archived, then new ones created).

**The guard on the third path.** A requester may not cancel a request while any work entry it produced
is validated (`WKE-045`). The refusal path carries no such guard; see the compatibility finding in
`WKE-044`.

**Worked outcome.** A one-day absence on Thursday 10 October had produced one absence row of eight
hours, and the day's attendance row had been archived. The approver refuses the request. Afterwards
the absence row is archived and cancelled, and a fresh eight-hour ordinary attendance row exists for
Thursday 10 October in the `draft` state. The originally archived attendance row stays archived: the
platform does not resurrect it, it generates a new one.

---

## 13. A contract ends, or its end date is brought forward

Performed by a human resources officer.

**Path A — the end date is written onto the version.** The write triggers the out-of-period removal of
[calculations.md, section 10.3](calculations.md#103-removal-outside-the-contract-period):

1. Every entry of the version dated after the new effective end date is found.
2. If any exists, the generated-to marker is pulled back to that date at the last representable
   microsecond and the entries are **deleted**.
3. If any of them is validated, the deletion is refused with "This work entry is validated. You can't
   delete it." and the whole write fails. The officer must first deal with the payroll period.

**Path B — the end date was already in force and generation runs over a wider period.** Step 5c of the
generation algorithm adds those entries to the nullifying condition instead, and they are archived
rather than deleted (see [calculations.md, section 10.2](calculations.md#102-nullifying-entries-beyond-the-version-end)).
Validated entries are excluded from the condition and survive.

**Path C — the version is removed.** Every non-validated entry of that version inside its effective
window is deleted (`WKE-040`). A version holding a validated entry cannot be removed at all.

**Records changed:** Work Entry; Employee Version.

---

## 14. An employee moves to a new schedule in the middle of a month

Performed by a human resources officer. This is the procedure that exercises the version clamping most
sharply.

1. Create a new Employee Version with the new working schedule, dated at the day the change takes
   effect.
2. The employee's earlier version now has an effective end date of the day before, because the
   effective end is the earlier of the day before the next version's version date and the contract end
   date.
3. Run generation over the month, from the calendar or by hand. For each version:
   - the earlier version is clamped to end at the end of its effective end date, and the entries
     beyond it — already generated under the old schedule — are added to the nullifying condition and
     archived;
   - the later version is clamped to start at its effective start date, and produces rows from that
     date on under the new schedule.
4. The day on which the new version starts belongs entirely to the new version.

**Worked outcome.** January of a year in which there are twenty-three working days. A forty-hour
schedule from the first; a thirty-five-hour schedule from the tenth. After regeneration the month
holds twenty-three rows totalling one hundred and sixty-eight hours: seven days at eight hours and
sixteen days at seven hours. Adding a third version from the twentieth, back on the forty-hour
schedule, gives twenty-three rows totalling one hundred and seventy-eight hours: seven at eight, six
at seven and ten at eight.

---

## 15. Operating across companies and time zones

1. Generation groups the versions by the pair of company and time zone before doing anything
   (`WKE-058`).
2. Each group is generated with that company as the acting company, so that the Working Time
   Exclusions read are that company's, or global ones.
3. Each group's requested period is converted into a window in that group's zone. Two employees in
   different zones asked for "1 September to 30 September" are generated over different windows on the
   universal time scale, and that is correct: each sees their own local month.
4. Every produced row is dated in its own group's zone.
5. The daily job deliberately processes one company per run and re-triggers itself for the rest.

**Worked outcome.** Two employees, one in Brussels and one in Hong Kong, both on a five-day schedule,
both asked for 1 August to 2 August. The Brussels employee's window runs from 31 July 22:00 to 2
August 21:59:59.999999 on the universal scale; the Hong Kong employee's from 31 July 16:00 to 2 August
15:59:59.999999. Both receive two rows dated 1 August and 2 August. No row of either employee is dated
31 July, although both windows begin on that universal day.

---

## 16. Replacing a block of days from the calendar

Performed by a human resources officer, for the common correction "these six days were actually
training".

1. In the calendar, select a block of day cells.
2. The multiple-selection toolbar appears, offering: a delete button, a reset button, a "Set" button
   that opens a small creation form, and one shortcut button per recently used kind.
3. **Delete** removes every selected non-validated entry.
4. **Set** opens a form with a kind, a duration and a description, and creates one entry per selected
   day with those values.
5. **Replace**, offered inside the same form, does the same and then deletes the entries that were
   there before.
6. A **shortcut button** performs a quick replacement with that kind and no explicit duration:
   - for each selected day that already holds entries, one new entry is created carrying the kind and
     the **existing total duration of that day**, and the old entries are deleted;
   - for each selected day that holds nothing, a forced generation of that single day is run, and the
     kind is written onto the first entry it produced.
7. **Reset** hands the selected days, as pairs of an employee and a date, to the regeneration operation
   in its slot mode (`WKE-052`), which collapses them into runs of consecutive days and forces one
   generation per run.
8. In all cases the days that hold a validated entry are excluded from the selection before anything
   happens.

**Records changed:** Work Entry.

---

## 17. Handing the day book to payroll

Performed by a payroll capability. This domain's part of the procedure is small and is listed so that
the boundary is unambiguous.

1. Generation must have reached the period. A payroll capability calls the employee-level entry point
   for its period before reading.
2. The capability reads, for each employee and each date of the period: the payroll code of the kind,
   the duration in hours, the pay rate stored on the entry and the extra-hours flag of the kind.
3. It calls the validation operation; if it reports failure the period cannot be closed and the
   conflicts must be resolved first.
4. It computes gross amounts. Nothing of that computation belongs to this domain; see
   [accounting-effects.md](accounting-effects.md).

The one additional service this domain offers payroll is the per-kind absence total: given one
employee and two instants, it widens them to whole days, finds every non-cancelled entry of that
employee in the range that is linked to a request in the validated state, groups them by the
request's absence kind and returns the total hours per kind.
