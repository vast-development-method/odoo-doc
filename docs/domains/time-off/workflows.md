# Time Off — Workflows

Every operational procedure of the domain, end to end: the actors, the preconditions, the
numbered steps with their decisions and branches, the records created or changed at each step
with the values written, the notifications and scheduled activities produced, the failure
conditions, and the postconditions.

The approval state machines themselves — the states, the permitted transitions per role and
per approval ladder, and the exact refusal messages — are in
[state-machines.md](state-machines.md). Every rule cited as `TOF-…` is in
[business-rules.md](business-rules.md).

---

## 1. Configure a Time Off Type

**Actor**: Time Off Administrator.
**Precondition**: the actor holds the Administrator group.

1. Open the Time Off Types catalogue and start a new record.
2. Enter the name, choose the request unit — Day, Half-Day or Hours — and choose whether the
   type counts as Absence or as Worked Time.
3. Choose the approval ladder for requests and, when the type requires an allocation, the
   approval ladder for allocations.
4. Name the Time Off Officers to notify. The list offers only non-portal users who hold the
   Officer group and belong to the acting company. The field is hidden when the request ladder
   is "None needed" or "By Employee's Approver" and the type does not use an officer ladder for
   allocations either.
5. Decide whether an allocation is required. When it is, decide whether employees may request
   additional allocation themselves.
6. Configure the behaviour flags: public-holiday inclusion, dashboard visibility, supporting
   document requirement, accrual eligibility, request stacking — offered only on a Worked Time
   type — and calendar event creation.
7. When a negative balance is allowed, enable the negative cap and enter the maximum excess
   amount, which must be at least one.
8. With the payroll companion package installed, choose the work entry type that absences of
   this kind produce.
9. With the attendance companion package installed, decide whether approving an absence of
   this kind consumes the employee's compensable extra hours.
10. With the Indian localization package installed, decide whether bridging days are included
    in the duration and whether the type is limited to Optional Holidays.
11. Choose the colour and the cover image.
12. Save.

**Records written**: one Time Off Type.
**Postconditions**: the type appears in the catalogue ordered by sequence, and becomes
selectable on a new request either immediately, when it requires no allocation, or as soon as
a valid allocation exists for the requester.
**Guards**: `TOF-030` through `TOF-037`.

---

## 2. Submit a Time Off Request

**Actor**: any internal user for themselves; a Time Off Approver for an employee they approve;
an Officer or an Administrator for anybody.
**Precondition**: the actor has an employee record, or names one they are allowed to name.

1. Open the new request form. The defaults are computed:
   - the employee is the actor's own employee record;
   - the requested start and end dates are today;
   - the Time Off Type is the first usable type by sequence, per
     [entities.md, section 4.5](entities.md#45-field-table--type-and-configuration-mirrors);
   - the day-period pair is Morning and Afternoon.

   When the caller supplies absolute instants instead of requested dates — which happens when
   the request is started by dragging on a calendar — those instants are converted into the
   actor's local dates, used as the requested dates, and dropped from the defaults.
2. Choose the Time Off Type. The selectable list contains every type that requires no
   allocation, plus every type that has a valid allocation for the chosen employee and either
   allows a negative balance or still shows a strictly positive provisionally remaining
   balance.
3. Choose the period.
   - Day-based type: enter the requested start date and the requested end date.
   - Half-day-based type: enter the dates and choose the start half and the end half.
   - Hour-based type: enter the dates and the start and end hours. The hours are pre-filled
     from the working schedule and are clamped into the closed intervals zero to 23.99 and
     zero to twenty-four.
4. The platform resolves the period into the absolute start and end in coordinated universal
   time with the algorithm of
   [calculations.md, chapter 3](calculations.md#3-from-request-dates-to-absolute-dates), then
   computes the duration in days and hours and the duration display with the algorithm of
   [chapter 4](calculations.md#4-the-duration-computation-algorithm), then refreshes the
   balance counters, the overlap warning, the rounding notice and the mandatory-day flag.
5. Optionally type a description and attach supporting documents.
6. Save.

On saving, the creation procedure runs:

7. When the type requires both approvals and the creation values name a state explicitly, the
   double-approval rule is checked (`TOF-046`).
8. When any request being created has no employee, the whole creation is rejected with
   *"There is no employee set on the time off. Please make sure you're logged in the correct
   company."*
9. The records are created without the usual creation entry in the discussion thread.
10. The duration is recomputed explicitly, because an automation rule reacting to the creation
    may otherwise persist a zero before the absolute dates have been derived.
11. The coverage check runs (`TOF-050` through `TOF-053`). A failure rolls the creation back.
12. The cached balance mirrors of every Time Off Allocation are invalidated.
13. For each created request, unless the creation runs in fast mode:
    - the employee's login user is subscribed as a follower;
    - when the request ladder is "By Employee's Approver", the employee's Time Off Approver is
      subscribed as well;
    - when the request ladder is "None needed", the request is approved immediately with
      elevated rights, the responsible approvers are subscribed, and the message *"The time off
      has been automatically approved"* is posted as a comment;
    - otherwise, and unless the creation comes from a file import, an approval activity is
      scheduled per [chapter 5](#5-scheduled-activities-and-notifications-by-transition).

**Records written**: one Time Off Request per employee; thread followers; either an approval
activity or, under the "None needed" ladder, everything produced by
[chapter 3](#3-approve-a-time-off-request).
**Postconditions**: the request is in state *To Approve*, or directly *Approved* under the
"None needed" ladder.
**Failure conditions**: `TOF-020` to `TOF-026`, `TOF-040`, `TOF-046`, `TOF-050` to `TOF-053`,
`TOF-090`, and the three database checks of
[entities.md, section 4.13](entities.md#413-database-constraints-and-validations).

---

## 3. Approve a Time Off Request

**Actor**: depends on the ladder and the current state; see
[state-machines.md, section 2.4](state-machines.md#24-the-same-map-as-tables).
**Precondition**: the request is in state *To Approve* or *Second Approval*, and the actor may
perform the transition.

1. The actor presses Approve — visible when the can-approve flag is true — or Validate —
   visible when the can-validate flag is true and the can-approve flag is false — on the form,
   in the list, on the card, or in the overview calendar.
2. Each selected request is routed to the first-approval set or to the validation set by the
   dispatch of
   [state-machines.md, section 2.11](state-machines.md#211-the-dispatch-of-the-approve-operation);
   a request that fits neither aborts the whole operation with *"You cannot approve this
   leave."*
3. Requests in the first-approval set are written to *Second Approval* with the first approver
   set to the acting user's employee record.
4. Requests in the validation set run the validation procedure of
   [state-machines.md, section 2.6](state-machines.md#26-side-effects-of-reaching-approved),
   whose materialisation step is detailed below.
5. Unless the operation runs in fast mode, the activity update of
   [chapter 5](#5-scheduled-activities-and-notifications-by-transition) runs for every selected
   request.

### 3.1 Materialisation of an approved absence

1. **Timesheet lines** — timesheet companion package, executed first. For every request whose
   employee is active:
   - read the employee's company internal project and its time off task; skip the request when
     either is missing or when the type counts as Worked Time;
   - when the working schedule has flexible hours and the request starts and ends on the same
     day, produce a single day entry whose hours are: the width of the hour range for an
     hour-based request, half the schedule's hours per day for a half-day request whose two
     halves are the same, otherwise the schedule's hours per day;
   - otherwise produce one entry per working day between the absolute instants, taken from the
     working schedule and ignoring the request's own Working Time Exclusion;
   - delete any timesheet lines previously generated for these requests, after clearing their
     absence link;
   - create one Analytic Line per entry carrying the name *"Time Off (`<index starting at
     one>`/`<total>`)"*, the project, the task, the project's analytic account, the day's hours
     as the quantity, the employee's login user, the day, the absence link, the employee, and
     the task's company or, failing that, the project's company.
2. **Working Time Exclusion**. For every request with an employee, create one record with the
   values listed in
   [state-machines.md, section 2.6](state-machines.md#26-side-effects-of-reaching-approved).
3. **Calendar Events**. For every request whose type creates calendar events:
   - the actor's create permission on the Calendar Event entity is verified;
   - the values are grouped by the employee's login user and each group is created in that
     user's name with elevated rights, with attendee mail suppressed, video call suppressed and
     the active entity set to the request;
   - each event carries the name *"`<employee name>` on Time Off : `<duration display>`"*, a
     duration of the day figure multiplied by the schedule's hours per day or by eight when
     there is none, the description taken from the reasons, the login user, the start and stop,
     the all-day flag, the privacy value "Confidential", the user's time zone, no activities,
     and a record reference pointing at the request;
   - the all-day flag is true when the request is not half-day-based, or when it is
     half-day-based and runs from a Morning to an Afternoon; for an hour-based type it is true
     when the day figure rounded to one decimal place is at least one;
   - when the all-day flag is true the start and stop are the local wall-clock instants in the
     request's time zone, otherwise they are the absolute instants;
   - the employee's login user contact, or failing that the employee's work contact, is added
     as an attendee;
   - each created event is linked back onto its request.
4. **Notification**. For every request, the message *"Your `<type display name>` planned on
   `<absolute start converted into the request's time zone>` has been accepted"* is posted and
   sent to the employee's login user contact.
5. **Work entries** — payroll companion package, executed last. For every request:
   - for every Employee Version whose contract period overlaps the request period, and only
     when the request period intersects the window for which work entries have already been
     generated, work entry values are produced for the whole day range of the request and
     created;
   - the existing work entries of the same employees in the same window are compared with the
     new absence entries: those that extend beyond the absence lose their absence link unless
     they are already validated, and those entirely contained in the absence are archived
     unless they are already validated;
   - for a French part-time employee whose absence had its end date extended by the French
     rule, the gaps between the employee's schedule and the company schedule are additionally
     filled with work entries carrying the type's work entry type, one per company attendance
     interval on a date that carries no work entry yet.

**Postconditions**: the request is *Approved*; the employee's working schedule now contains
the absence, so every availability, payroll and planning computation sees it; the calendar
shows the event; the timesheet shows the hours; payroll shows the work entries.

---

## 4. Refuse, cancel, reopen and delete a Time Off Request

### 4.1 Refuse

**Actor**: per the map of
[state-machines.md, section 2.4](state-machines.md#24-the-same-map-as-tables).

1. When any selected request is not in state *To Approve*, *Second Approval* or *Approved*, the
   whole operation is rejected with *"Time off request must be confirmed or validated in order
   to refuse it."*
2. Before the state changes, the employee's Time Off Approver is notified for every request
   whose ladder is "By Employee's Approver and Time Off Officer" and whose state is *Second
   Approval* or *Approved*, and for every request whose ladder is "By Employee's Approver" and
   whose state is *Approved*.
3. Requests that were in *Second Approval* are written to *Refused* with the **first** approver
   set to the acting user's employee record; all the others are written to *Refused* with the
   **second** approver set.
4. Because the state leaves *Approved*, the write procedure removes the Working Time Exclusion
   before the new state is stored.
5. The Calendar Event is archived.
6. For every request whose employee has a login user, the refusal message is posted and sent.
7. The activity update removes the approval activities.
8. The timesheet lines of the request are unlinked and deleted, and any missing public-holiday
   timesheet lines for the period are regenerated.
9. The work entries linked to the request are archived and attendance work entries are
   regenerated for the affected days.
10. With the Indian localization package installed, the durations of the neighbouring absences
    are restated, because a refused absence no longer bridges to its neighbours.

### 4.2 Cancel

**Actor**: the owner of the request; an Officer on behalf of an owner when the request starts
in the past.

1. The actor presses Cancel, which opens the cancellation dialog with the request pre-filled.
2. The actor types a reason and confirms.
3. When the request may not be cancelled, the operation is rejected with *"This time off cannot
   be cancelled."*
4. The reason is posted on the request's thread as an internal note.
5. The responsible approvers are determined from the ladder and the state held before the
   cancellation:
   - ladder "By Employee's Approver" and state *Approved*, or ladder "By Employee's Approver and
     Time Off Officer" and state *Second Approval*: the employee's Time Off Approver;
   - ladder "By Time Off Officer" and state *Approved*: the Officers named on the type;
   - ladder "By Employee's Approver and Time Off Officer" and state *Approved*: both.

   They are notified with the subject *"Cancelled Time Off"*.
6. The state is written to *Cancelled* with elevated rights.
7. The activities are removed, the Calendar Event is archived and the Working Time Exclusion is
   removed.
8. The timesheet lines are unlinked and deleted and the missing public-holiday timesheet lines
   are regenerated.
9. The work entries are archived and attendance work entries are regenerated.
10. A success notification is shown reading *"Your time off has been cancelled."*

**Postconditions**: the request is *Cancelled* and immutable. Only an Administrator may write
on it, and no state change is possible at all.

### 4.3 Reopen an approved request

**Actor**: an Officer.

1. The actor presses Back to Approval, visible only when the reopening permission is true,
   which requires the state to be *Approved* and the transition to *To Approve* to be permitted.
2. The state is written to *To Approve*. Because the state leaves *Approved*, the Working Time
   Exclusion is removed by the write procedure.
3. The activities are rescheduled as for a new request in *To Approve*.
4. The Calendar Event is archived and the Working Time Exclusion is removed again for safety.
5. The work entries are archived and attendance work entries are regenerated.

Records that may not be reopened are silently skipped rather than raising an error.

### 4.4 Delete

1. A user who is not an Officer may delete only requests in state *To Approve*, *Second
   Approval* or *Cancelled*, and only when the request does not start before today; otherwise
   *"Oops! `<state label>` Time-Off requests can only be deleted by Administrators."* or *"You
   can't delete a time off request that is in the past."*
2. An Officer who is not an Administrator may delete only requests in state *Cancelled* or *To
   Approve*; otherwise the same first message, this time printing that record's own state label.
3. An Administrator may delete a request in any state.
4. Before the deletion, the Calendar Event is archived and the Working Time Exclusion is
   removed with elevated rights; the timesheet lines are unlinked and deleted and the missing
   public-holiday timesheet lines are regenerated; the allocation balance mirrors are
   invalidated.
5. The deletion itself runs with the overlap check suppressed.

---

## 5. Scheduled activities and notifications by transition

### 5.1 Determining the responsible approvers

For a Time Off Request or a Time Off Allocation:

1. When the ladder is "By Employee's Approver", or the ladder is "By Employee's Approver and
   Time Off Officer" and the record is in state *To Approve*: the responsible is the employee's
   Time Off Approver; when that is empty, the login user behind the employee's hierarchical
   parent; when that is empty, the Officers named on the type.
2. When the ladder is "By Time Off Officer", or the ladder is "By Employee's Approver and Time
   Off Officer" and the record is in state *Second Approval*: the responsible is the set of
   Officers named on the type.
3. In every other case there is no responsible and no activity is created.

### 5.2 Activities on a Time Off Request

| State reached | Activity behaviour |
|---|---|
| `confirm` | When the type's request ladder is not "None needed", one "Time Off Approval" activity is created for each responsible user, with the note *"New `<type name>` Request created by `<creator name>`"* and a deadline of the absolute start date minus the activity type's delay of fifteen days, floored at today. When the request has no absolute start the deadline is today. |
| `validate1` | The "Time Off Approval" activity is marked done. One "Time Off Second Approve" activity is created for each responsible user, with the note *"Second approval request for `<type name>`"* and a deadline of the absolute start date — that activity type carries no delay — floored at today. |
| `validate` | Both activity types are marked done. |
| `refuse`, `cancel` | Both activity types are removed, including activities a user created by hand. |

```formula
deadline = ( absolute start date − delay count × delay unit of the activity type ) as a date
deadline = today                     , when the request has no absolute start
deadline = today                     , when the computed deadline is before today
```

### 5.3 Activities on a Time Off Allocation

| State reached | Activity behaviour |
|---|---|
| `confirm` | When the type's **request** ladder is not "None needed", one "Allocation Approval" activity is created for each responsible user, with the note *"New Allocation Request created by `<creator name>`: `<days rounded to two decimal places>` Days of `<type name>`"*. |
| `validate1` | The "Allocation Approval" activity is marked done and one "Allocation Second Approval" activity is created for each responsible user, with the note *"Second approval request for `<type name>`"*. |
| `validate` | Both activity types are marked done. |
| `refuse` | The "Allocation Approval" activity is removed. |

The condition on the **request** ladder rather than on the allocation ladder is recorded as a
**compatibility finding**: an allocation whose own ladder demands an approval receives no
approval activity when the type's request ladder is "None needed". A corrected behaviour would
test the allocation ladder; a compatible rebuild reproduces the observed test, because
approval remains possible from the allocation screens.

### 5.4 Discussion-thread notifications

| Event | Message and recipients |
|---|---|
| Automatic approval at creation | Comment "The time off has been automatically approved" on the request's thread. |
| Approval | "Your `<type display name>` planned on `<absolute start in the request's time zone>` has been accepted", sent to the employee's login user contact. |
| Refusal | "Your `<type display name>` planned on `<absolute start>` has been refused", sent to the employee's login user contact, only when the employee has a login user. |
| Refusal of a request that was *Second Approval* or *Approved* under a two-approval ladder, or *Approved* under an approver ladder | A separate notification to the employee's Time Off Approver, subject "Refused Time Off", body "`<request display name>` has been refused." |
| Cancellation with a reason | "The time off request has been cancelled for the following reason:" followed by the reason, posted as an internal note when the employee cancels and as a comment when the platform cancels; plus a notification to the responsible approvers, subject "Cancelled Time Off", body "`<request display name>` has been cancelled for the following reason: `<reason>`". |
| Public-holiday change giving days back | "Due to a change in global time offs, you have been granted `<number>` day(s) back." |
| Public-holiday change taking days | "Due to a change in global time offs, `<number>` extra day(s) have been taken from your allocation. Please review this leave if you need it to be changed." |
| Public-holiday change invalidating the request | "Due to a change in global time offs, this leave no longer has the required amount of available allocation and has been set to refused. Please review this leave." |
| First run of an accrual plan on an allocation | Internal note "This allocation have already ran once, any modification won't be effective to the days allocated to the employee. If you need to change the configuration of the allocation, delete and create a new one." |
| Employee departure shortening a request | "End date has been updated because the employee will leave the company on `<departure date>`." |
| Employee departure cancelling a request | The cancellation reason is "The employee will leave the company on `<departure date>`." and the responsible approvers are not notified. |
| Employee departure shortening an allocation | "Validity End date has been updated because the employee will leave the company on `<departure date>`." |
| Automatic cancellation by the invalid-absence job | The cancellation reason is "the accruated amount is insufficient for that duration." posted as an internal note. |
| Absence split by an Employee Version change | The new request's thread carries an origin link back to the request it was split from. |

The notification subtype used when a request reaches *Approved* is the type's request
notification subtype, falling back to the shipped "Time Off" subtype. The subtype used when an
allocation reaches *Approved* is the type's allocation notification subtype, falling back to
the shipped "Allocation Request" subtype. Every such subtype is automatically mirrored onto the
Department entity, which lets a user follow a whole department's absences.

Followers may be added to a request or an allocation in state *Second Approval* or *Approved*
only with elevated rights, because the record rules would otherwise forbid the write; the
platform checks read access first and then performs the subscription with elevated rights.

---

## 6. Request and approve a Time Off Allocation

**Actor**: an employee for themselves, on a type that allows employee requests; an Approver for
an employee they approve; an Officer or an Administrator for anybody.

1. Open the new allocation form. The defaults are: the actor's own employee record, today as the
   validity start, one day as the amount, the state *To Approve*, and the first usable type.
2. Choose the type, the amount and the validity period. For an hour-based type the amount is
   typed in hours and converted into days by dividing by the employee's hours per day on the
   validity start date.
3. To build an accrual allocation, choose an Accrual Plan. Setting a plan switches the
   allocation type to Accrual and zeroes the amount; the form then replays the plan from the
   validity start date up to the earlier of the validity end date and today, which shows the
   amount the employee would already have accrued.
4. Save. On creation:
   - a state other than *To Approve* in the creation values is rejected with *"Incorrect state
     for new allocation"*;
   - the department is filled from the employee when the caller supplies none;
   - the accrual cursor is initialised, see
     [entities.md, section 5.12](entities.md#512-initialisation-of-the-accrual-cursor-at-creation);
   - the employee's login user is subscribed; under the "By Time Off Officer" ladder the login
     user of the employee's hierarchical parent and the employee's Time Off Approver are
     subscribed as well;
   - unless the creation comes from a file import, the approval activity is scheduled;
   - when the ladder is "None needed", the approve operation runs immediately.
5. The approver presses Approve or Validate. The routing, the messages and the approver fields
   are in [state-machines.md, chapter 3](state-machines.md#3-the-allocation-state-machine).
6. Once *Approved*, the allocation contributes to the employee's balance from its validity
   start date and, for an accrual allocation, the daily accrual run starts advancing it.

**Failure conditions**: `TOF-060` to `TOF-064`, `TOF-091`, `TOF-092`.

---

## 7. Generate Time Off for multiple employees

**Actor**: a Time Off Approver, restricted to the "By Employee" mode, or a Time Off Officer.

1. Open the Multiple Requests dialog from the request list or from the card header.
2. Choose the Time Off Type, the population mode and the period, and optionally a description.
3. Confirm. When the actor is not an Officer and the mode is not "By Employee", the operation is
   rejected with *"As Time Off Responsible, you can only use the allocation mode 'By
   Employee'."*
4. The population is resolved:
   - By Employee: the listed employees, or, when the list is empty, every employee the actor may
     select;
   - By Employee Tag: the employees carrying the tag, limited to the allowed companies;
   - By Company: every employee of the chosen company;
   - By Department: the members of the chosen department.
5. The period is converted into instants: the start day at zero hours, zero minutes, zero
   seconds and the end day at twenty-three hours, fifty-nine minutes, fifty-nine seconds and
   999999 microseconds, both interpreted in the time zone of the chosen company's working
   schedule, falling back to the actor's time zone and then to coordinated universal time.
6. The conflicting requests are found: every request of those employees, in a state other than
   *Cancelled* and *Refused*, whose absolute start is at or before the new end and whose
   absolute end is strictly after the new start.
   - When any conflicting request uses an hour-based type, the whole operation is rejected with
     the message of `TOF-096`, followed by one bullet line per conflicting request display name.
   - Conflicting requests that span exactly one day are refused.
   - The remaining conflicting requests are split around the new period: each is shortened to
     end the day before the new start, and a new request is created starting the day after the
     new end, both keeping the original state. A resulting fragment whose absolute start is not
     strictly before its absolute end is discarded.
7. For every employee in the population, the working days between the two instants are computed
   from their working schedule. Employees with zero working days in the period are skipped.
8. One Time Off Request per remaining employee is created carrying the description, the type,
   the absolute instants, the requested dates, the number of working days as the day figure, the
   employee, and the state *Approved* when the actor is an Officer or the type's ladder is "None
   needed", otherwise *To Approve*. The creation runs with tracking, activity automation, the
   calendar synchronisation and the state check suppressed, and with the absolute instants taken
   as given rather than recomputed.
9. The materialisation procedure of [section 3.1](#31-materialisation-of-an-approved-absence) is
   run on the created requests, which materialises them whatever the state written.
10. The dialog returns a list of the generated requests.

**Postconditions**: one request per employee who has working days in the period; the conflicting
requests refused or split.

---

## 8. Generate Allocations for multiple employees

**Actor**: a Time Off Approver, "By Employee" mode only, or a Time Off Officer.

1. Open the New Group Allocation dialog.
2. Choose the type, the amount, the allocation kind — Regular Allocation or Based on Accrual
   Plan — the plan when applicable, the validity dates, the population mode and the reasons.
3. Confirm. The same allocation-mode restriction as in [chapter 7](#7-generate-time-off-for-multiple-employees)
   applies, with the same message.
4. The population is resolved exactly as in [chapter 7](#7-generate-time-off-for-multiple-employees),
   step 4.
5. For every employee an allocation is prepared carrying the description, the type, the amount —
   converted from hours by dividing by the employee's working schedule hours per day, falling
   back to the company schedule and then to eight hours — the employee, the state *To Approve*,
   the allocation kind, the validity dates, the plan and the reasons.
6. The allocations are created with forced asynchronous notification and with activity
   automation suppressed.
7. When the typed amount is zero and the allocations are accrual-based, each group sharing a
   validity end date is reset — last call set to the validity start, next call cleared, every
   amount zeroed, the pre-granted flag cleared, the expiry date and the expiring pool cleared —
   and then replayed by the accrual engine up to the earlier of the validity end date and today.
8. Allocations whose ladder is neither "None needed" nor "By Time Off Officer" are approved.
   Allocations whose ladder is "By Time Off Officer" are approved as well when the actor is an
   Officer.
9. The dialog returns a list of the generated allocations.

---

## 9. Create, change or delete a public holiday

**Actor**: a Time Off Officer or an Administrator.

A public holiday is a Working Time Exclusion with no resource. It applies to a single working
schedule when one is named, and to every schedule of the company otherwise.

1. The actor creates, edits or deletes one or more such records.
2. On creation, when the record names a working schedule, carries no resource, supplies both
   instants, and the actor's time zone differs from the schedule's time zone, the instants are
   reinterpreted per
   [entities.md, section 11.2](entities.md#112-time-zone-reinterpretation-at-creation).
3. The overlap constraint is checked: two public holidays of the same company whose periods
   intersect and whose schedules are compatible are refused with *"Two public holidays cannot
   overlap each other for the same working hours."*
4. The company of the record is derived: the company of the employee behind the absence link,
   else the company of the working schedule, else the acting company.
5. The affected absences are collected: every Time Off Request in a state other than *Refused*
   and *Cancelled* whose employee's company matches a changed record's company and whose period
   strictly overlaps a changed record's period — the request ending strictly after the holiday
   starts and starting strictly before the holiday ends. The previous durations and the previous
   states are remembered.
6. Every affected request is moved to *To Approve* with elevated rights, which removes its
   Working Time Exclusion, and its duration and duration display are recomputed against the new
   holiday landscape.
7. For each affected request:
   - when the duration went down and the type requires an allocation, the message is *"Due to a
     change in global time offs, you have been granted `<difference>` day(s) back."*;
   - when the duration went up and the type is not the shipped Sick Time Off type, the message is
     *"Due to a change in global time offs, `<difference>` extra day(s) have been taken from your
     allocation. Please review this leave if you need it to be changed."*;
   - the previous state is restored with elevated rights, bypassing the approval matrix, and the
     coverage check is re-run;
   - when the coverage check fails, the request is refused instead and the message becomes *"Due
     to a change in global time offs, this leave no longer has the required amount of available
     allocation and has been set to refused. Please review this leave."*;
   - when a message was produced, it is posted on the thread and sent to the employee's login
     user contact, or failing that to the employee's work contact, with the subject *"Your Time
     Off"*.
8. Every request that ended up *Approved* has its Working Time Exclusion recreated.
9. With the attendance companion package installed, every attendance overlapping the changed
   period is re-evaluated for overtime.
10. With the timesheet companion package installed, public-holiday timesheet lines are created
    for future holidays and removed when the holiday disappears.

---

## 10. Change an employee's working schedule through an Employee Version

**Actor**: a human resources user.

When an Employee Version is created or changed in a way that touches the contract start date,
the contract end date, the version date or the working schedule, the absences of that employee
are restated.

### 10.1 Creating a version

1. Collect the absences of the employee that are neither *Refused* nor *Cancelled*, that end on
   or after the new contract start date, that begin on or before the new contract end date when
   one is given, and that use a different working schedule.
2. For each such absence, remember the state it currently holds, then:
   - refuse the absence when its requested start date is before the new contract start date;
   - otherwise move it back to *To Approve*.

   An absence already in state *Refused* or *To Approve* is left as it is.
3. Create the version.
4. Recompute the versions whose contract period overlaps the absence.
   - When they all carry the same working schedule, no split happens: the absence adopts that
     schedule and, unless it is hour-based, its absolute instants are recomputed with the
     overlap and state checks suppressed; when it is still in state *Approved* at that moment it
     is revalidated. Because step 2 has already moved it to *To Approve* or *Refused*, on this
     path it is in practice not revalidated and the employee must have it approved again.
   - When they carry more than one distinct working schedule, the absence is split in step 5.
5. Create one new Time Off Request per overlapping version, each running from the later of the
   original requested start date and the version's contract start date, to the earlier of the
   original requested end date and the version's contract end date, an absent contract end date
   counting as infinitely far away. Every new request except the one belonging to the last
   overlapping version is created in the state remembered in step 2; the one belonging to the
   last overlapping version is created in state *To Approve*. A fragment whose absolute start is
   not strictly before its absolute end is discarded. **The original request is not deleted**:
   it stays in the state step 2 put it into, which for a request that began before the new
   contract start date is *Refused*.
6. The fragments are created with tracking, activity automation and the state check suppressed;
   fragments in state *Approved* are materialised; each fragment's thread receives an origin link
   back to the request it came from.
7. When creating the fragments raises a coverage error, the whole operation is rejected with
   *"Changing the contract on this employee changes their working schedule in a period they
   already took leaves. Changing this working schedule changes the duration of these leaves in
   such a way the employee no longer has the required allocation for them. Please review these
   leaves and/or allocations before changing the contract."* followed by a blank line, the words
   *"This error has been triggered by:"* and the underlying message.

### 10.2 Writing on a version

The same procedure runs with two differences of ordering:

1. The version is written first and the overlapping versions are recomputed afterwards, so step
   4 runs while the absence still holds its original state. An absence that adopts a single new
   schedule and is *Approved* **is** revalidated, which rebuilds its Working Time Exclusion, its
   Calendar Event, its timesheet lines and its work entries against the new schedule.
2. Only when more than one distinct schedule overlaps is the absence's state remembered and the
   absence refused, whatever its start date, before the fragments are created.

### 10.3 Changing the schedule directly on the employee

Every **future** request of the employee whose schedule differs from the new one is rewritten to
the new schedule, its absolute instants are recomputed — except for an hour-based request, whose
clock hours the user chose explicitly — with the overlap and state checks suppressed, and every
such request that is *Approved* is revalidated, which rebuilds its Working Time Exclusion, its
Calendar Event, its timesheet lines and its work entries. When the recomputed durations no
longer fit the entitlement, the whole write is rejected with *"Changing this working schedule
results in the affected employee(s) not having enough leaves allocated to accomodate for their
leaves already taken in the future. Please review this employee's leaves and adjust their
allocation accordingly."*

### 10.4 Hour-based allocations

Whenever the working schedule changes, every hour-based allocation of the employee has its day
amount restated from its stored hour amount divided by the employee's new hours per day.
Without this, the next accrual would revalue the hours accrued under the old schedule.

**Guard**: after everything, the constraint of `TOF-025` still forbids an absence spanning
versions with different schedules.

---

## 11. Register an employee departure

**Actor**: a human resources user, through the departure dialog.

1. The departure date is recorded on the employee by the
   [human resources core](../human-resources-core/README.md) domain.
2. Every Time Off Request of the departing employees whose absolute end is strictly after the
   departure date is collected and split into two groups: those that start on or before the
   departure date, and those that start after it.
3. Requests that start on or before the departure date are split at the day after the departure
   date, which shortens the original to end on the departure date and creates a new request for
   the remainder.
4. Requests that were genuinely shortened — their end now on or before the departure date —
   receive the comment *"End date has been updated because the employee will leave the company on
   `<departure date>`."*
5. Every request that starts after the departure date, every shortened request that was not
   actually shortened, and every newly created remainder are collected. Those in state *Approved*
   or *Second Approval* are cancelled with the reason *"The employee will leave the company on
   `<departure date>`."* and without notifying the responsible approvers. The rest are deleted
   with the state check suppressed.
6. Every Time Off Allocation of the departing employees with no validity end date, or with a
   validity end date strictly after the departure date, is collected. Those whose validity starts
   after the departure date are deleted with the allocation state check suppressed. The others
   receive the comment *"Validity End date has been updated because the employee will leave the
   company on `<departure date>`."* and their validity end date is set to the departure date.

---

## 12. The daily accrual run

**Actor**: the scheduled process "Accrual Time Off: Updates the number of time off", every day.

1. Select every Time Off Allocation where the allocation type is `accrual`, the state is
   *Approved*, an Accrual Plan is set, an employee is set, the validity end date is empty or
   strictly after the present instant, and the next call date is empty or on or before today at
   midnight.
2. Run the accrual engine on that selection with a target date of today, with logging on and
   without forcing a period.

The engine is specified in [accrual-plans.md, chapter 7](accrual-plans.md#7-the-engine).

---

## 13. The daily invalid absence run

**Actor**: the scheduled process "Time Off: Cancel invalid leaves", every day.

1. Collect every Time Off Request whose absolute start falls between today at zero hours and the
   day thirty-one days from today at twenty-three hours, fifty-nine minutes, fifty-nine seconds
   and 999999 microseconds, and whose state is *To Approve*, *Second Approval* or *Approved*,
   ordered by absolute start **descending**.
2. Collect the accrual allocations of those employees, for those types, in state *Approved*,
   whose validity overlaps the same window.
3. Keep only the requests whose type has at least one such accrual allocation, and re-sort them
   by absolute start descending.
4. For each remaining request, in that order:
   - compute the balance data of the employee for the type at the request's start date;
   - when the total allocated amount is zero, cancel the request with the reason *"the accruated
     amount is insufficient for that duration."*, posted as an internal note, without notifying
     the responsible approvers;
   - otherwise compute the total provisional excess; the allowed excess is the type's maximum
     excess amount when the type allows a negative balance and zero otherwise; when the excess is
     at or below the allowed excess the request is kept, and otherwise it is cancelled with the
     same reason.

Processing the requests from the furthest in the future to the nearest is deliberate: cancelling
the furthest request first frees entitlement for the nearer ones, so the earliest absences
survive.

---

## 14. Approve or refuse from an electronic mail notification

**Actor**: any authenticated user who received the notification.

1. The notification message embeds three links for a request — approve, validate, refuse — and
   two for an allocation — validate, refuse — each carrying the record identifier and a signed
   token.
2. Following a link authenticates the user, verifies the token against the record, and performs
   the corresponding operation.
3. When the operation raises any error, the user is redirected to a generic fallback page instead
   of seeing the error, and the record is unchanged.
4. In every case the user ends on the record's own page.

---

## 15. Print the sixty-day summary

**Actor**: a Time Off Officer.

1. Select one or more employees and launch the Time Off Summary dialog.
2. Choose the start date, defaulting to the first day of the current month, and the state filter:
   Approved, Confirmed, or Both Approved and Confirmed.
3. Confirm. The printed document is produced in landscape on a custom page of two hundred and ten
   by two hundred and ninety-seven millimetres, with a top margin of thirty millimetres, a bottom
   margin of twenty-three millimetres and side margins of five millimetres.
4. The grid covers exactly sixty consecutive days from the start date. Saturdays and Sundays are
   shaded. Each employee is one row; each day of an absence in the chosen states is painted with
   the colour of its Time Off Type. The row total is the sum of the day figures of the absences
   that intersect the window. A legend lists the type names and their colours. When the document
   is launched from a department selection the rows are grouped by department and each group
   repeats the day band.

---

## 16. Cross-domain effects of an absence

This chapter collects, in one place, everything that changes outside this domain when an absence
reaches or leaves the *Approved* state.

| Consumer | Effect of reaching *Approved* | Effect of leaving *Approved* |
|---|---|---|
| [Attendances and working time](../attendances-and-working-time/README.md) | A Working Time Exclusion is written on the employee's resource, so every work-interval, availability and planning computation stops counting those hours. | The exclusion is removed and the hours reappear. |
| [Calendar and scheduling](../calendar-and-scheduling/) | A Calendar Event is created when the type asks for one, marked confidential, with the employee as attendee and no video call. | The event is archived. |
| [Work entries](../work-entries/README.md) | Work entries carrying the type's work entry type are created for the absence days; attendance work entries fully inside the absence are archived and those extending beyond it lose their absence link. | The absence work entries are archived and attendance work entries are regenerated for those days. |
| [Timesheets](../timesheets/README.md) | One Analytic Line per working day is created on the company's internal project and time off task, unless the type counts as Worked Time. | The lines are unlinked and deleted, and missing public-holiday lines are regenerated. |
| Attendance overtime | A validated absence of a type that deducts extra hours consumes the employee's compensable extra hours; a public-holiday change re-evaluates the overtime of the overlapping attendances. | The consumption is released. |
| [Messaging and activities](../messaging-and-activities/README.md) | Approval activities are marked done and the approval message is posted under the type's notification subtype. | Activities are removed or rescheduled and the refusal or cancellation message is posted. |
| [Identity and access](../identity-and-access/README.md) | While the absence covers the present instant the employee's login user and contact show an on-leave online status and a return date. | The status returns to its base value. |

Nothing in this chapter writes to the general ledger; see
[accounting-effects.md](accounting-effects.md).

---

## 17. Reconciliation notes

1. **Where the state machines live.** One draft carried the approval matrices inside this file
   and the other carried them in a dedicated file. They now live in
   [state-machines.md](state-machines.md), and this file references them; no row was dropped in
   the move.
2. **Validation of a request.** One draft described the materialisation as a single step of the
   approval, the other as a numbered procedure with the timesheet lines first and the work
   entries last. The ordering is load-bearing, because the timesheet generation must not see the
   absence's own Working Time Exclusion, so the numbered procedure of
   [section 3.1](#31-materialisation-of-an-approved-absence) is kept.
3. **Cross-domain effects.** One draft listed them only as a capability line in its overview.
   They are collected here as [chapter 16](#16-cross-domain-effects-of-an-absence).
4. **French work entries.** Neither draft specified the gap filling performed for a French
   part-time employee. It is part of the packages in scope and is recorded in
   [section 3.1](#31-materialisation-of-an-approved-absence), step 5.
