# Time Off — Business Rules

This file lists every validation, constraint, invariant, error message, permission check,
locking rule and edge-case behaviour of the domain. Error messages are reproduced exactly
as the system produces them, with placeholders rendered in words.

---

## 1. Invariants

These statements must hold at every commit boundary. A re-implementation that breaks one
of them will diverge in behaviour even when every individual rule below is honoured.

| # | Invariant |
|---|---|
| I1 | A Time Off Request's absolute start and end are **always** derived from the request layer against the request's working schedule and time zone. They are never accepted as user input; a write that supplies them is rewritten into the request layer. |
| I2 | A request's duration in days and in hours is **always** derived from the absolute dates, the working schedule and the applicable public holidays. There is no way to type a duration. |
| I3 | Working time is removed from an employee's availability **only** through a Working Time Exclusion record. A request that is approved without such a record having been created is a corrupt state. |
| I4 | An entitlement is usable **only** while its allocation is in state approved. Requests already charged against an allocation that leaves that state become excess. |
| I5 | The stored amount of an allocation (`number_of_days`, the number of days) is a **gross** figure that includes the part already consumed. Every cap and every carry-over truncation must therefore add the consumed part back after truncating. |
| I6 | An allocation's amount is stored in **days** whatever the type's request unit; the hours figure is derived, and is re-derived whenever the employee's hours per day changes. |
| I7 | The accrual cursor moves forward only. Replaying the processing with the same target date must not grant anything a second time. |
| I8 | A request may not span two employment terms with **different** working schedules. |
| I9 | Two requests of the same employee whose types do not allow requests on top may not overlap, in any state other than cancelled or refused. |
| I10 | A request's company always equals its employee's company (falling back to the department's, then to the acting company). The multi-company record rules depend on this. |

---

## 2. Rules on the Time Off Request

### 2.1 Creation requires an employee

Creating a request with no employee is refused:

> There is no employee set on the time off. Please make sure you're logged in the correct company.

This is a user error, not a database constraint, and it is raised before any record is
written even when only one entry of a batch lacks an employee.

### 2.2 Date ordering

Two database check constraints:

| Rule | Message |
|---|---|
| the absolute start must be at or before the absolute end | The start date must be before or equal to the end date. |
| the request start date must be at or before the request end date | The request start date must be before or equal to the request end date. |

### 2.3 Non-negative duration

A database check constraint requires the duration in days to be greater than or equal to
zero:

> If you want to change the number of days you should use the 'period' mode

### 2.4 Overlap

A request whose state is neither refused nor cancelled and whose type does not allow
requests on top may not overlap another request of the same employee, of a type that also
does not allow requests on top, in a state other than cancelled or refused. Overlap is
**strict**: the start of one must be strictly before the end of the other and vice versa,
so two requests that merely touch at an instant do not conflict.

The message is the overlap warning text built in
[Entities, section 4.5](entities.md#45-the-overlap-warning) — beginning either

> You've already booked time off which overlaps with this period:

when every conflicting request belongs to the acting user, or

> An employee already booked time off which overlaps with this period:

otherwise, followed by one line per conflict.

The check runs whenever the absolute start, the absolute end, the employee or the state
changes. It is skipped when the calling context carries the date-check suppression flag,
which the internal split routine and the deletion path both set.

### 2.5 Deleting a request

Two ladders, by group:

**An ordinary employee (not an officer).** Each record must satisfy both:

- its state must be one of awaiting approval, second approval or cancelled, otherwise:

  > Oops! `<state label>` Time-Off requests can only be deleted by Administrators.

  where the state label printed is that of the **first** record of the batch;
- its absolute start date must not be in the past, otherwise:

  > You can't delete a time off request that is in the past.

**An officer who is not an administrator.** Every record whose state is not cancelled or
awaiting approval is refused with the same "only be deleted by Administrators" message,
this time printing that record's own state label.

**An administrator.** No state restriction.

Deleting always runs the cancellation side effects first: the meeting is deactivated and
the working time exclusion is removed, in an elevated context.

### 2.6 Modifying a request

**Non-officers.** When the write touches anything other than the three attachment fields
(`attachment_ids`, `supported_attachment_ids`, `message_main_attachment_id`):

- if any record has already begun (its absolute start date is before today), the acting
  user is not that employee's absence approver, and the state is neither awaiting approval
  nor draft:

  > You must have manager rights to modify/validate a time off that already begun

- if any record is cancelled:

  > Only a manager can modify a canceled leave.

**Everyone.** A request in state second approval or approved may not have its absolute
dates or its employee changed at all:

> This modification is not allowed in the current state.

This check is skipped when the calling context carries the state-check suppression flag,
which the split routine, the multi-employee generation wizard and the employment-term
re-scheduling all set.

### 2.7 Duplication

> A time off cannot be duplicated.

raised unless every record in the batch is cancelled or refused, or unless the internal
copy-check suppression flag is set.

### 2.8 The employment-term constraint

A request may not span employment terms carrying different working schedules:

> A leave cannot be set across multiple versions with different working schedules.
>
> Please create one time off for each version period.
>
> Time off:
> `<the request's display name>`
>
> Versions:
> - '`<version name or employee name>`' from `<start date>` to `<end date, or the word "undefined">`

with one bullet per overlapping term. The check runs on every change of the absolute
dates. It is a *creation-time* guarantee only: a term created or amended **after** the
request already exists can retroactively put a request across two schedules, which is why
the employment-term write procedure splits requests rather than relying on this check (see
[Workflows, chapter 12](workflows.md#12-changing-an-employment-term)).

### 2.9 Zero-duration approval

Approving a request that has an employee and a duration of zero days aborts the whole
batch:

> The following employees are not supposed to work during that period:
> `<comma-separated employee names>`

In the presence of the work-entry integration this rule is narrowed: requests whose type
maps to a work entry type whose code is `LEAVE110`, `LEAVE210` or `LEAVE280` are exempt and
may be approved with a zero duration.

### 2.10 Entitlement validity

The entitlement check runs on creation, and on any write that touches the request start
date, the absolute dates, the type, the employee or the state — except when the new state
is refused or cancelled.

Requests are grouped by (type, absolute start date). For each group whose type requires an
allocation:

**When the type does not allow a negative balance:**

1. Compute the balance data for the employees at the group's date.
2. Compute the same data again with the requests of this group **ignored**.
3. For each employee:
   - if the maximum allowed is zero (no entitlement at all):

     > You do not have any allocation for this time off type.
     > Please request an allocation before submitting your time off request.

   - if neither computation recorded any excess, the employee passes;
   - if the two excess maps differ **and** the new one has at least as many entries as the
     old one:

     > `<employee name>` does not have a valid allocation for the leave type `<type name>` to cover that request.

   The comparison of the two maps, rather than a plain "is the balance negative" test, is
   what lets an employee *reduce* an already over-consuming request without being blocked.

**When the type allows a negative balance:**

1. If every request in the group is cancelled or refused, the group passes.
2. If the maximum allowed is zero, the "You do not have any allocation…" message is raised.
3. If the provisionally remaining balance is strictly below the negative of the maximum
   excess amount:

   > `<employee name>` does not have a valid allocation for the leave type `<type name>` to cover that request.

**Worked example of the exceeding message.** An employee holds one approved allocation of
**two days** of the type `Limited`, valid for the calendar year. The employee requests the
first to the fourth of February (four working days). The balance data reports a maximum of
two, a provisionally remaining balance of minus two and an excess entry under the fourth of
February. The previous computation (ignoring this request) recorded no excess, so the maps
differ and the new one is larger. The creation is refused with, exactly:

> David Employee does not have a valid allocation for the leave type Limited to cover that request.

A subsequent, valid request of two days is accepted; **modifying** it afterwards to five
days raises the same message.

### 2.11 Mandatory days

After the entitlement check, and only for a user who does **not** hold the Time Off Officer
group:

> You are not allowed to request time off on a Mandatory Day

raised when any request in the batch intersects a mandatory day applicable to its employee
(see [Entities, section 8.1](entities.md#81-applicability-rule)).

### 2.12 Overtime-deductible types

When the attendance integration is present, a type may be marked *Deduct Extra Hours*. For
such a type that does **not** require an allocation, a request is only valid while the
employee's unspent compensable overtime stays non-negative:

```formula
unspent = ( approved compensable overtime hours )
          − ( hours of requests of overtime-deductible, allocation-free types
              in any state other than refused or cancelled )
          − ( hours of allocations of overtime-deductible types
              in state awaiting approval, second approval or approved )
```

The check runs on creation, on any write touching the duration, the request dates, the
state, the employee or the type, on reset to the approval queue and on approval. When the
balance is negative:

- if the request's employee is the acting user's own:

  > You do not have enough extra hours to request this leave

- otherwise:

  > The employee does not have enough extra hours to request this leave.

For an allocation of an overtime-deductible type the same balance check applies on creation
and on any write touching the amount or the type:

> The employee does not have enough overtime hours to request this leave.

and, additionally, a non-officer may not edit the amount of such an allocation once it has
left the awaiting-approval state:

> Only an Officer or Administrator is allowed to edit the allocation duration in this status.

### 2.13 Work entry locking

When the work-entry integration is present, creating or writing a request wraps the
operation in an error-checking window spanning from one day before the earliest request
start date to one day after the latest request end date (the one-day margin covers time
zone shifts, because the absolute dates are not yet computed at that moment). Inside that
window the work-entry domain refuses the change when it would conflict with validated work
entries or a closed payroll period. The check is skipped when the write touches none of the
employee, the state and the two request dates.

A request that has produced at least one **validated** work entry can no longer be
cancelled: the can-cancel flag is forced to false for it.

---

## 3. Rules on the Time Off Type

### 3.1 Changing the allocation requirement

> The allocation requirement of a time off type cannot be changed once leaves of that type have been taken. You should create a new time off type instead.

Raised whenever the allocation-requirement flag is written and at least one request of the
type exists. The check is skipped while shipped data is being installed. Additionally, a
data-load write that would set the flag to the value it already holds drops that value from
the write entirely, so that re-loading shipped data never trips the rule.

### 3.2 The negative cap

Database check constraint:

> The maximum excess amount should be greater than 0. If you want to set 0, disable the negative cap instead.

### 3.3 Requests on top

> You cannot allow requests on top of leaves of type 'Absence'.

Only a type whose kind of time off is "Worked Time" may allow overlapping requests.

### 3.4 Accrual eligibility of worked time

> leaves of type 'Worked Time' should be always eligible for accrual rate.

A type whose kind of time off is "Worked Time" may not clear the accrual-eligibility flag.
The flag's default follows the kind: true for "Worked Time", false for "Absence".

### 3.5 Freezing the public-holiday inclusion flag

> You cannot modify the 'Public Holiday Included' setting since one or more leaves for that                         time off type are overlapping with public holidays, meaning that the balance of those employees would be affected by this change.

(The message is produced with the literal run of whitespace shown, resulting from the way
it is written in the source.) It is raised when the flag is written and any request of the
type, whose absolute start falls inside the current calendar year, in state awaiting
approval, second approval or approved, overlaps by date with a resource-less working time
exclusion of the type's company or of the acting company.

### 3.6 Deleting a type

There is no explicit guard. Deletion is prevented in practice because requests and
allocations reference the type with a required link, so the database refuses the delete
while any exist. The intended retirement path is archiving.

---

## 4. Rules on the Time Off Allocation

### 4.1 Creation state

> Incorrect state for new allocation

raised when a creation supplies any state other than awaiting approval.

### 4.2 Amount

Database check constraint: the amount must be strictly greater than zero **when the
allocation type is regular**; an accrual allocation may legitimately start at zero.

> The duration must be greater than 0.

### 4.3 Validity window

> The Start Date of the Validity Period must be anterior to the End Date.

raised when a non-empty end date precedes the start date.

### 4.4 Reducing an allocation below what has been consumed

On any write that touches the amount (in days or in hours) or the state, the platform
measures the **non-provisional** excess before and after the write:

```formula
excess_before = sum of the amounts of the excess entries that are not provisional, before the write
excess_after  = sum of the amounts of the excess entries that are not provisional, after the write
```

and, when the excess has grown and the type either forbids a negative balance or the new
excess exceeds the maximum excess amount:

> You cannot reduce the duration below the duration of leaves already taken by the employee.

### 4.5 Deleting an allocation

Two independent guards:

> You cannot delete an allocation request which is in `<state label>` state.

raised for any allocation whose state is neither awaiting approval nor refused. This guard
is bypassed when the calling context carries the allocation state-check suppression flag,
which only the departure procedure sets.

> You cannot delete an allocation request which has some validated leaves.

raised when the type requires an allocation and the taken figure is greater than zero.

### 4.6 Approving one's own allocation

> Only a time off Administrator can approve/refuse their own requests.

raised when the allocation's employee is the acting user's own employee, the type's
allocation validation mode is not "None needed", and the acting user does not hold the
Time Off Administrator group.

### 4.7 Duplication

A copy is silently forced back to state awaiting approval; there is no error.

---

## 5. Rules on the Accrual Plan and its levels

| Rule | Message |
|---|---|
| The start count must be strictly positive when the milestone is reached "After", and exactly zero when it is reached "At allocation creation" | You can not start an accrual in the past. |
| The rate must be strictly greater than zero | You must give a rate greater than 0 in accrual plan levels. |
| A limited carry-over on a level that carries unused accruals over must set a strictly positive maximum | You cannot have a maximum quantity to carryover set to 0. |
| A carried-over validity must set a strictly positive count | You cannot have an accrual validity time set to 0. |
| A yearly cap must set a strictly positive yearly maximum | You cannot have a cap on yearly accrued time without setting a maximum amount. |
| A running cap must set a maximum strictly greater than zero | You cannot have a balance cap on accrued time set to 0. |
| The weekly frequency requires a weekday | Weekday must be selected to use the frequency weekly |
| The twice-a-month frequency requires the first day to be strictly less than the second | The first day must be lower than the second day. |
| An unrecognised frequency | Your frequency selection is not correct: please choose a frequency between theses options:Hourly, Daily, Weekly, Twice a month, Monthly, Twice a year and Yearly. |
| Deleting a plan linked to a live accrual allocation | Some of the accrual plans you're trying to delete are linked to an existing allocation. Delete or cancel them first. |

When the attendance integration adds the **Per Hour Worked** frequency, one more rule
applies:

> You can't base accrued time on hours worked, because time is accrued at the start of the period.

and, correspondingly, switching a plan to grant at the start of the period silently
converts a "Per Hour Worked" level into an "Hourly" one.

### 5.1 Derived constraints (forced values rather than errors)

| Situation | Forced value |
|---|---|
| The plan does not allow carry-over | every level loses unused accruals |
| A level loses unused accruals | its carry-over option becomes unlimited and its carried-over validity is cleared |
| A level does not cap the running balance | its maximum becomes zero |
| The plan restricts a type | every level's rate unit becomes Days for a Day or Half-Day type, Hours for an Hour type |
| The plan does not restrict a type | every level after the first copies the first level's rate unit |
| The plan grants at the start of the period | the plan is not based on worked time |
| A level's start count is zero | its milestone is "At allocation creation" |
| A level's milestone is "At allocation creation" | its start count becomes zero |
| A day-of-month selection exceeds the length of its month | it is clamped to the length of that month, using a leap year as the reference so that the twenty-ninth of February stays selectable |

---

## 6. Rules on Working Time Exclusions

### 6.1 Overlapping public holidays

> Two public holidays cannot overlap each other for the same working hours.

Raised when two records **without a resource**, belonging to the same company, overlap by
date range, and either share a working schedule or at least one of them has no schedule.

### 6.2 Date ordering

> The start date of the time off must be earlier than the end date.

### 6.3 End date defaulting

When the end is empty or is not strictly after the start, it is set to the last second of
the start's local day (twenty-three hours, fifty-nine minutes, fifty-nine seconds) in the
reading user's zone, or in the company schedule's zone when the reader has none.

### 6.4 Company derivation

```formula
company = the linked request's employee's company
company = the schedule's company              , when there is no linked request
company = the acting company                  , when neither is available
```

---

## 7. Permission checks

### 7.1 Groups

| Group | Implies | Meaning |
|---|---|---|
| Internal User | — | Any employee. May file their own requests and see their own entitlement. |
| Time Off Responsible | Internal User | Someone who is the designated absence approver of at least one employee. Granted automatically. |
| Officer: Manage all requests | Time Off Responsible, Human Resources Officer | Sees and manages every request and allocation. |
| Administrator | Officer | Configures types, accrual plans and mandatory days; may do anything. |

### 7.2 Automatic membership of the Responsible group

- When an employee is created with an absence approver, that user is added to the
  Responsible group.
- When an employee's absence approver is changed, the new approver is added to the group if
  not already in it, and the **previous** approvers are then cleaned: a previous approver
  who is no longer the absence approver of any employee is removed from the group.
- The same cleaning runs after every login-user creation.

### 7.3 Who may act on a request

See the reachable-state map in
[State Machines, section 2.3](state-machines.md#23-the-reachable-state-map). Summarised:

| Action | Ordinary employee | Absence approver | Officer | Administrator |
|---|---|---|---|---|
| Create for self | yes | yes | yes | yes |
| Create for another | no | only for employees they approve | yes | yes |
| First approval, mode "both" | no | yes | yes | yes |
| Final approval | no | only when the mode is "By Employee's Approver" | yes | yes |
| Refuse | no | only when the mode is not "By Time Off Officer" | yes | yes |
| Send an approved request back to the approval queue | no | no | yes | yes |
| Cancel own request | yes, unless it has already begun | yes | yes | yes |
| Re-open a refused or cancelled request | no | no | yes | yes |
| Delete | only own, in a permitted state, not in the past | — | only cancelled or awaiting approval | any |

### 7.4 The double-validation rule

Independent of the map, and skipped for administrators:

- moving to second approval: a **non-officer** who is not the absence approver of an
  employee concerned is refused with

  > You cannot first approve a time off for `<employee name>`, because you are not his time off manager

- moving to approved: a non-officer is refused with

  > You don't have the rights to apply second approval on a time off request

### 7.5 The description privacy rule

The description of a request is readable in clear only by:

- a holder of the Time Off Officer group;
- the login user of the request's employee;
- the absence approver of the request's employee.

Everyone else reads the five characters `*****`. Writing the description is silently
ignored for everyone else. Searching on the description is rewritten to search the private
column, restricted to the searcher's own requests unless the searcher is an officer.

### 7.6 The multi-employee wizards

> As Time Off Responsible, you can only use the allocation mode 'By Employee'.

raised on both generation wizards when a non-officer chooses any mode other than "By
Employee".

### 7.7 Approving from the calendar report

> You are not allowed to approve this leave request.
>
> You are not allowed to refuse this leave request.

raised when the acting user is neither an officer nor the employee's absence approver for a
type whose request validation mode is "By Employee's Approver" or "both".

### 7.8 Subscription to the discussion thread

Adding a follower or a mention to a request or allocation in state second approval or
approved is performed in an elevated context after a read-access check, because the record
rules would otherwise forbid the write.

---

## 8. Record rules

### 8.1 Time Off Request

| Rule | Groups | Operations | Domain |
|---|---|---|---|
| Own requests, read | Internal User | read | the request's employee's login user is the reader |
| Own or approved-for, create and write | Internal User | create, write | (the employee's login user is the actor **and** the state is neither approved nor second approval) **or** (the type's request validation mode is one of "By Employee's Approver", "both", "None needed" **and** the employee's absence approver is the actor) |
| Own, delete | Internal User | delete | the employee's login user is the actor **and** the state is awaiting approval or second approval |
| Approved-for, read | Time Off Responsible | read | the employee's absence approver is the actor |
| Approved-for, create and write | Time Off Responsible | create, write | (the employee's login user is the actor **and** the state is not approved) **or** the employee's absence approver is the actor |
| All, read | Officer | read, write, create, delete | unrestricted |
| Officer create and write | Officer | create, write | (the employee's login user is the actor **and** the state is not approved) **or** the employee's login user is not the actor or is unset — in effect, an officer may not write on their **own** approved request |
| Administrator | Administrator | all | unrestricted |
| Multi-company | global | all | the request's company is one of the reader's allowed companies |

### 8.2 Time Off Allocation

| Rule | Groups | Operations | Domain |
|---|---|---|---|
| Own or approved-for, read | Internal User | read | the employee's absence approver is the actor **or** the employee's login user is the actor |
| Own or approved-for, create and write | Internal User | create, write | (the employee's login user is the actor **and** the state is awaiting approval) **or** (the allocation validation mode is one of "By Employee's Approver", "both", "None needed" **and** the employee's absence approver is the actor) |
| Own drafts, delete | Internal User | delete | the employee's login user is the actor **and** the state is `draft` — a value the allocation state machine never takes, so this rule grants nothing in practice |
| Approved-for, create and write | Time Off Responsible | create, write | (the employee's login user is the actor **and** the state is not approved) **or** the employee's absence approver is the actor |
| All, read | Officer | read | unrestricted |
| Officer, all operations | Officer | all | (the employee's login user is the actor **and** the state is not approved) **or** the employee's login user is not the actor or is unset |
| Administrator | Administrator | all | unrestricted |
| Multi-company | global | all | (the allocation has no employee **or** the employee's company is allowed) **and** the type's company is allowed or empty |

### 8.3 Other entities

| Entity | Rule |
|---|---|
| Working Time Exclusion | Internal Users may read every record but not write, create or delete; officers may do everything. |
| Time Off Type | Global multi-company rule: the type's company is allowed, **or** the type has no company and its country is empty or belongs to an allowed company. |
| Accrual Plan | Global multi-company rule on the plan's company, allowing empty. |
| Mandatory Day | Global multi-company rule on the company, allowing empty. |
| Time Off Analysis | Internal Users may read only the rows for which the department-manager access flag is true; officers may read everything. A global multi-company rule applies. |
| Time Off Calendar Report | A global multi-company rule applies. |

---

## 9. Access rights matrix

| Entity | Internal User | Time Off Responsible | Officer | Administrator |
|---|---|---|---|---|
| Time Off Request | create, read, update, delete | (inherits) | create, read, update, delete | create, read, update, delete |
| Time Off Allocation | create, read, update, delete | (inherits) | create, read, update, delete | create, read, update, delete |
| Time Off Type | read | (inherits) | read | create, read, update, delete |
| Accrual Plan | — | — | read | create, read, update, delete |
| Accrual Plan Level | — | — | read | create, read, update, delete |
| Mandatory Day | read | (inherits) | (inherits) | create, read, update, delete |
| Working Time Exclusion | (from the working-time domain) | — | create, read, update, delete | (inherits) |
| Calendar Meeting | (from the calendar domain) | — | create, read, update, delete | (inherits) |
| Meeting Attendee | — | — | create, read, update, delete | (inherits) |
| Meeting Type | — | — | — | create, read, update, delete |
| Activity Type | — | — | — | create, read, update, delete |
| Time Off Analysis | read | (inherits) | (inherits) | (inherits) |
| Time Off Calendar Report | read | (inherits) | (inherits) | (inherits) |
| Time Off by Employee and Type | — | — | — | read, update |
| Time Off Summary Wizard | — | — | create, read, update | (inherits) |
| Cancel Time Off Wizard | create, read, update, delete | (inherits) | (inherits) | (inherits) |
| Generate Time Off Wizard | — | create, read, update, delete | (inherits) | (inherits) |
| Generate Allocations Wizard | — | create, read, update, delete | (inherits) | (inherits) |

Access rights are cumulative through group implication: Officer implies Time Off
Responsible and Human Resources Officer; Administrator implies Officer.

---

## 10. Locking rules

| Lock | Rule |
|---|---|
| Approved and second-approval states | The absolute dates and the employee of a request may not change; the modification error is raised. |
| Requests that have begun | A non-officer who is not the employee's absence approver may not modify a request whose start date has passed, unless it is still awaiting approval. |
| Cancelled requests | Only a manager may modify a cancelled request; every other modification is refused. |
| Validated work entries | A request that produced a validated work entry can no longer be cancelled. |
| Payroll periods | The work-entry integration blocks creation and modification of requests that fall inside a period whose work entries are validated or locked. |
| Consumed allocations | The amount of an allocation may not be reduced below what has already been consumed by approved requests. |
| Used types | The allocation requirement of a type may not change once any request of the type exists. |
| Overlapping public holidays | The public-holiday inclusion flag of a type may not change while requests of that type overlap public holidays in the current year. |
| Company country | A company's country may not change while requests or allocations exist for its employees whose type is restricted to a different country. The message is: *"The company country cannot be changed while time off leaves or allocations with the country exist."* This check is suppressed while the automated test suite is running. |

---

## 11. Automatic cancellation of invalid absence

A daily job protects accrual-driven entitlements from being over-consumed by requests
placed further in the future than the accrual can support. Its rule is:

1. Consider the window from today to today plus thirty-one days.
2. Select requests whose absolute start falls in that window and whose state is awaiting
   approval, second approval or approved, ordered by absolute start **descending**.
3. Select the accrual allocations of those employees and types, in state approved, valid
   over the window.
4. Keep only the requests whose type appears among those allocations, re-sorted by absolute
   start descending.
5. For each such request, compute the balance data at the request's start date:
   - if the maximum allowed is zero, force-cancel the request;
   - else, let the excess limit be the type's maximum excess amount when the type allows a
     negative balance and zero otherwise; if the total provisional excess is at or below
     that limit, leave the request alone; otherwise force-cancel it.

The cancellation reason posted is:

> the accruated amount is insufficient for that duration.

posted as a **note** (not a comment), which also means the responsible parties are notified
per the normal cancellation procedure.

Processing the requests from the **latest** start date backwards is deliberate: cancelling
the furthest request first frees entitlement for the nearer ones, so the earliest requests
survive.

---

## 12. Mandatory days

A mandatory day blocks absence for the employees it applies to. The applicability rule is
in [Entities, section 8.1](entities.md#81-applicability-rule). The block applies only to
users who do **not** hold the Time Off Officer group; an officer may create absence for an
employee on a mandatory day.

The per-request flag used by the interface to show the warning uses a slightly narrower
domain than the applicability rule: it additionally requires the mandatory day's working
schedule to be empty or to equal the **request's** working schedule (rather than the
employee's), and when the type carries a company it requires the mandatory day to belong to
that company.

---

## 13. Edge cases

### 13.1 A request placed entirely on non-working time

Created successfully with a duration of zero days and zero hours; cannot be approved
(see [section 2.9](#29-zero-duration-approval)).

### 13.2 A public holiday declared over an existing request

Handled by the re-evaluation procedure of
[Workflows, chapter 11](workflows.md#11-declaring-a-public-holiday), which recomputes the
duration, informs the employee of the change, and refuses the request if the new duration
no longer fits the entitlement.

### 13.3 A working schedule changed on an employee

Every **future** request of the employee whose schedule differs from the new one is
rewritten to the new schedule, its absolute dates are recomputed (except for hourly
requests, whose clock hours the user chose explicitly), and the working time exclusions of
the approved ones are recreated. When the recomputed durations no longer fit the
entitlement, the whole write is refused with:

> Changing this working schedule results in the affected employee(s) not having enough leaves allocated to accomodate for their leaves already taken in the future. Please review this employee's leaves and adjust their allocation accordingly.

### 13.4 An employment term created or amended across an existing request

See [Workflows, chapter 12](workflows.md#12-changing-an-employment-term). When the split
produces requests whose durations no longer fit, the operation is refused with:

> Changing the contract on this employee changes their working schedule in a period they already took leaves. Changing this working schedule changes the duration of these leaves in such a way the employee no longer has the required allocation for them. Please review these leaves and/or allocations before changing the contract.
>
> This error has been triggered by:
> `<the underlying validation message>`

### 13.5 A department or approver change on an employee

- Changing the employee's parent sets the absence approver to the parent's login user, but
  **only** for employees whose current approver is the previous parent's login user or who
  have no approver at all.
- Changing the department or the parent rewrites the department on every request of the
  employee that is either awaiting approval or starts in the future, and on every
  allocation of the employee that is awaiting approval; the manager is rewritten on those
  allocations as well.

### 13.6 An archived employee

Archiving an employee empties the absence approver link on it. Requests and allocations
remain; the employee link is marked as restricting deletion, so the employee record itself
cannot be deleted while any exist.

### 13.7 Multi-company request creation

The company of a request follows its employee. When a user acting for company A creates a
request for an employee of company B, the record lands in company B and the multi-company
record rule may immediately hide it from the creator; the platform offers to switch to the
type's company.

### 13.8 An allocation with no end date placed before one with an end date

The consumption order (allocations with an end date first, ascending) means an unlimited
allocation is always consumed last, whatever its start date. An employee holding both a
year-limited grant and an unlimited one therefore burns the expiring one first.

### 13.9 An accrual allocation whose plan has no levels

Silently skipped by the processing; the balance never moves.

### 13.10 An accrual allocation whose plan has not started

When the target date is before the first level's start date and the cursor has never run,
the allocation is skipped entirely; the cursor stays empty so that a later run retries.

### 13.11 An accrual level whose cap has already been exceeded

The grant amount becomes negative (the running-cap formula subtracts the current balance
from the capped total) and the balance therefore **decreases**. This is the behaviour; a
re-implementation must not clamp the grant at zero.

### 13.12 A carry-over cut-off that coincides with the target date

The cut-off function compares strictly, so a reference date equal to the cut-off keeps that
year's cut-off rather than moving to the next. The closest-expiry computation compensates by
adding a year when the two coincide, because days accrued **on** the cut-off belong to the
following carry-over period.

### 13.13 A half-day request on a duration-based schedule

The proportional day figure is rounded to the nearest half day. On any other schedule it is
left at its computed value, which on an irregular schedule can be a figure such as one
point five or one.

### 13.14 An hourly request whose window contains no working time

Duration zero hours, duration display `0:00 hours`. Creatable, not approvable.

### 13.15 A fully flexible employee

Has no working schedule. Duration is measured on the wall clock; hours per day is taken as
twenty-four; public holidays are still subtracted, by whole local days.

### 13.16 Requests that start after the balance reference date

Excluded from the ordinary charging pass and handled by the deferred set, so that a request
placed in December is not charged against an accrual balance that will only exist in
December. The shortfall is reported as the *exceeding duration*, a negative number, rather
than as an excess entry.
