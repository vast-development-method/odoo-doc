# Time Off — Business Rules

Every validation, constraint, invariant, permission check, locking rule and edge-case
resolution of the domain, numbered with one stable identifier scheme. The prefix is `TOF-`
and the number is stable within this file; other files of this folder cite rules by that
number. Every message is reproduced exactly as the system produces it, with the placeholders
described in words.

Three role words are used throughout, with the meanings fixed in
[state-machines.md, chapter 1](state-machines.md#1-the-approval-ladders):

- **Officer** — the acting user holds the Time Off Officer group, which the Time Off
  Administrator group implies.
- **Administrator** — the acting user holds the Time Off Administrator group.
- **Approver** — the acting user is named as the Time Off Approver (`leave_manager_id`) on
  the employee record concerned.

An index of every rule is in [chapter 16](#16-index-of-rule-identifiers); the mapping from
the identifiers used by the two source drafts of this folder is in
[chapter 17](#17-mapping-of-former-rule-identifiers).

---

## 1. Invariants

These statements hold at every commit boundary. A rebuild that breaks one of them diverges in
behaviour even when every individual rule below is honoured.

| Rule | Invariant |
|---|---|
| **TOF-001** | A Time Off Request's absolute start and end (`date_from`, `date_to`) are **always** derived from the request layer against the request's working schedule and time zone. They are never accepted as user input; a write that supplies one of them is rewritten into a write of the corresponding requested date. |
| **TOF-002** | A request's duration in days (`number_of_days`) and in hours (`number_of_hours`) is **always** derived from the absolute dates, the working schedule and the applicable public holidays. There is no way to type a duration. |
| **TOF-003** | Working time is removed from an employee's availability **only** through a Working Time Exclusion (`resource.calendar.leaves`) record. A request that is *Approved* without such a record existing is a corrupt state. |
| **TOF-004** | An entitlement is usable **only** while its Time Off Allocation is in state *Approved*. Requests already charged against an allocation that leaves that state become excess. |
| **TOF-005** | The stored amount of an allocation (`number_of_days`, the number of days) is a **gross** figure that includes the part already consumed. Every cap and every carry-over truncation must therefore add the consumed part back after truncating. |
| **TOF-006** | An allocation's amount is stored in **days** whatever the type's request unit; the hour figure is derived from it, and is re-derived whenever the employee's hours per day changes. |
| **TOF-007** | The accrual cursor moves forward only. Replaying the processing with the same target date must not grant anything a second time. |
| **TOF-008** | A request may not span two Employee Versions carrying **different** working schedules. |
| **TOF-009** | Two requests of the same employee whose types do not allow requests on top may not overlap, in any state other than *Cancelled* or *Refused*. |
| **TOF-010** | A request's company always equals its employee's company, falling back to the department's company and then to the acting company. Every multi-company record rule depends on this. |
| **TOF-011** | Every balance figure exists in two forms: the plain form counts only *Approved* consumption, the provisional form counts *To Approve* and *Second Approval* consumption as well. Screens, the type-usability test and the coverage check all read the provisional form; only the reporting "taken" figure reads the plain form. |
| **TOF-012** | A Time Off Allocation has no cancelled state and is never archived; a Time Off Request is never archived. Withdrawal of a request is *Cancelled*, withdrawal of an allocation is *Refused* or deletion. |

---

## 2. The request period and its duration

**TOF-020. An employee is mandatory.** A creation whose values carry no employee is rejected,
before any record is written, even when only one entry of a batch lacks an employee:

> There is no employee set on the time off. Please make sure you're logged in the correct company.

**TOF-021. The resolved period is ordered.** A database check named `_date_check2` requires
the absolute start to be at or before the absolute end:

> The start date must be before or equal to the end date.

**TOF-022. The requested period is ordered.** A database check named `_date_check3` requires
the requested start date to be at or before the requested end date:

> The request start date must be before or equal to the request end date.

**TOF-023. The duration is not negative.** A database check named `_duration_check` requires
the duration in days to be greater than or equal to zero:

> If you want to change the number of days you should use the 'period' mode

**TOF-024. Overlap.** A request whose state is neither *Refused* nor *Cancelled*, and whose
type does not allow requests on top (`allow_request_on_top`), may not overlap another request
of the same employee, of a type that also does not allow requests on top, in a state other
than *Cancelled* or *Refused*. Overlap is **strict**: the start of one must be strictly before
the end of the other and the end of one strictly after the start of the other, so two requests
that merely touch at an instant do not conflict. The message is the overlap warning built in
[entities.md, section 4.11](entities.md#411-the-overlap-warning): the header

> You've already booked time off which overlaps with this period:

when the request has an employee and every conflicting request belongs to the reading user, or

> An employee already booked time off which overlaps with this period:

otherwise, then one line per distinct conflict, each preceded by a newline and a tabulation
character, reading *"`<employee name>` from `<start date>` to `<end date>` - `<state
label>`"*, with the employee name printed as the empty string in the first case. Identical
lines are printed once. The check runs whenever the absolute start, the absolute end, the
employee or the state changes, and is skipped entirely when the operation carries the
date-check suppression flag, which the internal split routine, the multi-employee generation
and the deletion path all set.

**TOF-025. A request may not span Employee Versions with different working schedules.**
Whenever the absolute dates are written, the Employee Versions whose contract period overlaps
the request period are collected. When they carry more than one distinct working schedule the
write is rejected with:

> A leave cannot be set across multiple versions with different working schedules.
>
> Please create one time off for each version period.
>
> Time off:
> `<the request's display name>`
>
> Versions:
> - '`<version name, or the employee name when the version has none>`' from `<contract start date>` to `<contract end date, or the literal word "undefined" when there is none>`

with one bullet line per overlapping version. This is a **creation-time** guarantee only: a
version created or amended *after* the request already exists can retroactively place a
request across two schedules, which is why the version write procedure of
[workflows.md, chapter 10](workflows.md#10-change-an-employees-working-schedule-through-an-employee-version)
splits requests instead of relying on this check.

**TOF-026. Payroll period locking.** With the payroll companion package installed, creating or
writing a request wraps the operation in an error-checking window running from one day before
the earliest requested start date to one day after the latest requested end date; the one-day
margin covers time-zone shifts, because the absolute dates are not yet computed at that
moment. Inside that window the work entry machinery refuses the change when it would conflict
with validated work entries or with a closed payroll period. The wrapper is skipped when the
write touches none of the employee, the state and the two requested dates.

**TOF-027. The resolved period is frozen after the first approval.** Writing the absolute
start, the absolute end or the employee of a request whose state is *Second Approval* or
*Approved* is rejected:

> This modification is not allowed in the current state.

The check is skipped when the operation carries the state-check suppression flag, which the
split routine, the multi-employee generation, the version re-scheduling and the departure
procedure all set.

**TOF-028. Modifying a request that has already begun.** A user who is not an Officer, writing
any field other than the three attachment fields (`attachment_ids`,
`supported_attachment_ids`, `message_main_attachment_id`), is rejected when any record in the
selection has an absolute start before today, the actor is not that employee's Approver, and
the state is not *To Approve*:

> You must have manager rights to modify/validate a time off that already begun

**TOF-029. A working schedule change may not invalidate existing absences.** When changing the
working schedule of an employee makes an already recorded future absence exceed the available
entitlement, the whole write is rejected with:

> Changing this working schedule results in the affected employee(s) not having enough leaves allocated to accomodate for their leaves already taken in the future. Please review this employee's leaves and adjust their allocation accordingly.

The same situation reached through an Employee Version is rejected with:

> Changing the contract on this employee changes their working schedule in a period they already took leaves. Changing this working schedule changes the duration of these leaves in such a way the employee no longer has the required allocation for them. Please review these leaves and/or allocations before changing the contract.
>
> This error has been triggered by:
> `<the underlying coverage message>`

Both messages are reproduced with the spelling they carry, including "accomodate".

---

## 3. The Time Off Type

**TOF-030. The negative cap carries an amount.** A database check named `check_negative`
requires the negative cap to be disabled or the maximum excess amount
(`max_allowed_negative`) to be strictly greater than zero:

> The maximum excess amount should be greater than 0. If you want to set 0, disable the negative cap instead.

**TOF-031. Request stacking is reserved for worked time.** Allowing requests on top on a type
whose kind of time off (`time_type`) is `leave`, labelled "Absence", is rejected:

> You cannot allow requests on top of leaves of type 'Absence'.

Only a type counted as Worked Time may be requested on top of another absence, and only such a
type is exempt from `TOF-024`.

**TOF-032. Worked time is always eligible for the accrual rate.** Clearing the accrual
eligibility flag (`elligible_for_accrual_rate`) on a type whose kind of time off is `other`,
labelled "Worked Time", is rejected:

> leaves of type 'Worked Time' should be always eligible for accrual rate.

The derived default sets the flag to true for a Worked Time type and to false for an Absence
type; on an Absence type an Administrator may set it back to true.

**TOF-033. The public-holiday inclusion flag is frozen while it would move balances.** Writing
`include_public_holidays_in_duration` is rejected when any request of the type whose absolute
start falls inside the current calendar year, in state *To Approve*, *Second Approval* or
*Approved*, overlaps by calendar date any resource-less Working Time Exclusion of the type's
company or of the acting company:

> You cannot modify the 'Public Holiday Included' setting since one or more leaves for that                         time off type are overlapping with public holidays, meaning that the balance of those employees would be affected by this change.

The message is reproduced with the literal run of whitespace it contains.

**TOF-034. The allocation requirement is frozen once the type has been used.** Writing
`requires_allocation` while at least one Time Off Request of the type exists is rejected:

> The allocation requirement of a time off type cannot be changed once leaves of that type have been taken. You should create a new time off type instead.

Two exceptions: the check does not run while shipped configuration records are being loaded,
and a write whose value equals the value already stored is dropped from the write before the
check, so that reloading the shipped catalogue never trips the rule.

**TOF-035. Country derivation.** Setting a non-empty company on a type recomputes the country
to that company's country. Clearing the company leaves the country unchanged. The selectable
countries are the countries of the acting user's allowed companies.

**TOF-036. Company and country are frozen once the type is used.** As soon as at least one
request or one allocation references the type, the company and the country become read-only on
the form. The country is additionally read-only whenever a company is set, because it is then
derived.

**TOF-037. Ordering and default selection.** The catalogue is ordered by `sequence` ascending.
When a query supplies no explicit ordering and an employee is present in the calling context,
the result is re-sorted in memory as specified in
[entities.md, section 3.2](entities.md#32-identity-ordering-and-display-name). The default
type proposed on a new request is the first type by sequence matching "requires no allocation
or has a valid allocation", preferring a type whose request unit is `hour` when the form
defaults already describe an hour-based request.

**TOF-038. Deleting a type.** There is no explicit guard. Deletion is prevented in practice
because Time Off Requests and Time Off Allocations reference the type through a link marked as
restricting deletion, so the database refuses the deletion while any such record exists. The
intended retirement path is archiving.

**TOF-039. Type usability.** A type is usable by an employee when it requires no allocation,
or when the employee holds at least one *Approved* allocation of that type whose validity
window covers the evaluated dates and which is either accrual-driven or carries a strictly
positive granted amount together with a provisionally remaining balance strictly above the
negative of the allowed excess, that allowance being zero when the type does not allow a
negative balance. Three progressively narrower tests exist and must not be unified; they are
enumerated in [entities.md, section 3.4](entities.md#34-two-different-tests-answer-is-this-type-usable).

---

## 4. Entitlement coverage

The coverage check is the validation that a Time Off Request is funded. Its arithmetic is the
balance consumption algorithm of
[calculations.md, chapter 6](calculations.md#6-the-balance-consumption-algorithm); the
decision procedure is in
[calculations.md, chapter 8](calculations.md#8-the-coverage-check).

**TOF-050. An allocation must exist.** For a request whose type requires an allocation, when
the employee's maximum allowed amount for that type, evaluated at the requested start date, is
zero, the operation is rejected with the two-line message:

> You do not have any allocation for this time off type.
> Please request an allocation before submitting your time off request.

This rule fires whether or not the type allows a negative balance.

**TOF-051. Coverage when a negative balance is allowed.** For a type that allows a negative
balance, and after `TOF-050` has passed, when the employee's provisionally remaining balance at
the group's date is strictly below the negative of the type's maximum excess amount, the
operation is rejected:

> `<employee name>` does not have a valid allocation for the leave type `<type name>` to cover that request.

The whole rule is skipped when every request in the evaluated group is already *Cancelled* or
*Refused*.

**TOF-052. Coverage when no negative balance is allowed.** For a type that does not allow a
negative balance, the balance data are computed twice: once including the requests being
checked and once with them ignored. When the two excess maps differ **and** the including map
holds at least as many entries as the excluding map, the operation is rejected with the same
message as `TOF-051`. When both maps are empty the request passes. Comparing the two maps,
rather than testing whether the balance is negative, is what lets an employee who is already
over-consuming reduce a request, or edit an unrelated one, without being blocked.

**Worked example.** An employee holds one *Approved* allocation of **two days** of the type
named "Limited", valid for the calendar year, and requests the first to the fourth of February,
four working days. The balance data report a maximum allowed of two, a provisionally remaining
balance of minus two, and one excess entry under the fourth of February. The computation that
ignores this request records no excess, so the maps differ and the new one is larger. The
creation is refused with, exactly:

> David Employee does not have a valid allocation for the leave type Limited to cover that request.

A subsequent request of two days is accepted; extending it afterwards to five days raises the
same message.

**TOF-053. When the coverage check runs.** After every creation of a request, and after every
write that touches the requested start date, the absolute start, the absolute end, the type,
the employee or the state, except when the newly written state is *Refused* or *Cancelled*. The
evaluated requests are grouped by the pair (type, requested start date) and each group is
evaluated independently. The check also runs at the end of a public-holiday restatement, where
its failure converts into a refusal of the request instead of a rejection of the operation
(`TOF-127`).

**TOF-054. Consumption ordering.** Within one employee and one type, absences consume the
*Approved* allocations in this order:

1. allocations that carry a validity end date, ascending by that end date, so that the
   entitlement expiring soonest is spent first;
2. then allocations with no validity end date whose kind is Accrual Allocation;
3. then allocations with no validity end date whose kind is Regular Allocation.

An allocation is skipped for a given absence when its validity start date falls after the
calendar date of the absence's end, or when it carries a validity end date falling before the
calendar date of the absence's start. Only the part of the absence falling inside the
allocation's validity window is charged; when that intersection is narrower than the absence,
the chargeable amount is recomputed from the employee's working schedule over the intersection
alone. The amount charged is capped by the allocation's provisionally remaining balance.

**TOF-055. Unit of the balance arithmetic.** For a type whose request unit is `day` or
`half_day` every balance figure is expressed in days and the absence duration used is the day
figure; for a type whose request unit is `hour` every figure is expressed in hours and the
duration used is the hour figure. An allocation's amount is converted with the employee's hours
per day for an hour-based type.

**TOF-056. Excess recording.** When an absence still carries an uncharged remainder after every
eligible allocation has been walked, and that remainder rounded to two decimal places is
strictly positive, an excess entry is recorded under the calendar date of the absence's end,
carrying the remaining amount, a flag saying whether the absence is still pending, and the
identity of the absence. Excess entries are what `TOF-051` and `TOF-052` compare.

**TOF-057. Pending consumption.** Restated from `TOF-011` for the coverage rules: the plain
figures count only *Approved* absences; the provisional figures additionally count absences in
state *To Approve* and *Second Approval*. Every guard in this chapter reads the provisional
figures.

**TOF-058. Future absences funded by accrual are deferred.** An absence that starts strictly
after the evaluation date, and for which at least one accrual allocation exists that is still
valid at the evaluation date and whose validity starts on or before the absence's end, is not
charged against any allocation at that evaluation date and produces no excess entry. It is set
aside and, after the charging loop, the whole deferred set is compared with the balance the
accrual will have reached on the latest of their start dates. The shortfall is published as the
*exceeding duration*, a number that is zero or negative.

**TOF-059. An allocation that is not approved funds nothing.** Only allocations in state
*Approved* are read by the consumption algorithm, archived employees included. An allocation
that leaves *Approved* turns every absence it was paying for into excess, which is what
`TOF-004` states as an invariant and what `TOF-063` guards against on a write.

---

## 5. Deleting, refusing, cancelling and reopening

**TOF-040. Every state write passes the guard ladder.** Before any state is written — including
a creation whose values name a state explicitly — the guard of
[state-machines.md, section 2.9](state-machines.md#29-the-guard-ladder-and-its-messages) for a
request, and of
[state-machines.md, section 3.6](state-machines.md#36-the-guard-ladder-and-its-messages) for an
allocation, is evaluated in order and raises the **first** message that applies. The platform
superuser bypasses the ladder entirely. The messages, in evaluation order for a request, are
"You can't do the same action twice.", "Not possible state. State Approve is only used for
leave needed 2 approvals", "A cancelled leave cannot be modified.", "You can only cancel your
own leave. You can cancel a leave only if this leave is approved, validated or refused.", "You
can't reset a leave. Cancel/delete this one and create an other", "Only a Time Off
Officer/Manager can approve a leave.", "You can't approve a validated leave.", "Only a Time Off
Officer/Manager can validate a leave.", "You can't approve this refused leave.", "You can only
validate a leave with validation by Time Off Manager.", "Only a Time Off Officer/Manager can
refuse a leave.", "You can't refuse a leave with validation by Time Off Officer." and finally
the platform's own access error when the target is permitted by the map but the record rules
forbid the write. Three consequences are worth stating: writing the state a record already
holds is always refused; moving to *Second Approval* under a ladder other than "By Employee's
Approver and Time Off Officer" is always refused; and a *Cancelled* request can never change
state again, whoever the actor is.

**TOF-041. Approve routing.** Pressing Approve on a request that the actor may neither validate
nor move to *Second Approval* aborts the whole operation:

> You cannot approve this leave.

The equivalent guard on an allocation is:

> Allocation must be "To Approve" in order to approve it.

**TOF-042. Zero-duration approval.** Validating a selection that contains a request with an
employee and a duration of zero days aborts the whole operation:

> The following employees are not supposed to work during that period:
> `<comma-separated employee names>`

The message is followed by a newline and a space before the list. With the payroll companion
package installed the rule is narrowed: requests whose type maps to a work entry type whose
code is `LEAVE110`, `LEAVE210` or `LEAVE280` — the three long-term absence codes — are exempt
and may be validated with a zero duration.

**TOF-043. Validation state check.** When the state check is active and any request in the
selection may not be validated, the whole validation is refused:

> You can't validate this leave.

**TOF-044. Refusal precondition.** Refusing a selection that contains a request whose state is
not *To Approve*, *Second Approval* or *Approved* aborts the whole operation:

> Time off request must be confirmed or validated in order to refuse it.

The equivalent guard on an allocation is:

> Allocation request must be confirmed, second approval or validated in order to refuse it.

**TOF-045. Cancellation precondition.** Cancelling a request whose computed cancellation
permission is false is rejected:

> This time off cannot be cancelled.

The permission is the entry for the target `cancel` in the reachable-state map: the request must
belong to the acting user and must not start before today, unless the actor is an Officer. With
the payroll companion package installed the permission is additionally forced to false when at
least one **validated** work entry points at the request, so such a request can no longer be
cancelled at all.

**TOF-046. The double-approval rule.** Independently of the reachable-state map, a record whose
ladder is "By Employee's Approver and Time Off Officer" passes an extra check at creation and on
every explicit state write. The check is skipped entirely for an Administrator.

- Moving to *Second Approval*: the employees concerned whose Approver is **not** the acting user
  are collected; when that set is not empty and the actor is not an Officer, the operation is
  rejected with *"You cannot first approve a time off for `<name of the first such employee>`,
  because you are not his time off manager"*.
- Moving to *Approved*: a non-Officer is rejected with *"You don't have the rights to apply
  second approval on a time off request"*.

**TOF-047. Reopening precondition.** Back to Approval is offered only when the state is
*Approved* and the map permits the target *To Approve*, which in practice means the actor is an
Officer. Records in the selection that may not be reopened are silently skipped rather than
raising an error.

**TOF-048. Deleting a request.** Three ladders apply.

- **A user who is not an Officer.** Each record must satisfy both conditions: its state is one
  of *To Approve*, *Second Approval* and *Cancelled*, otherwise *"Oops! `<state label>` Time-Off
  requests can only be deleted by Administrators."*, where the state label printed is that of
  the **first** record of the selection; and its absolute start must not be in the past,
  otherwise *"You can't delete a time off request that is in the past."*
- **An Officer who is not an Administrator.** Every record whose state is not *Cancelled* or *To
  Approve* is refused with the same "only be deleted by Administrators" message, this time
  printing that record's own state label.
- **An Administrator.** No state restriction.

A deletion always runs the reversal side effects first: the Calendar Event is archived and the
Working Time Exclusion removed with elevated rights, the timesheet lines are unlinked and
deleted, the missing public-holiday timesheet lines are regenerated, and the balance mirrors of
the allocations are invalidated. The deletion itself runs with the overlap check suppressed.

**TOF-049. Duplicating a request.** Duplication is rejected:

> A time off cannot be duplicated.

unless every record in the selection is *Cancelled* or *Refused*, or unless the internal
copy-check suppression flag is set, which only the internal split routine sets. Duplicating a
Time Off Allocation is always permitted; the copy is silently forced back to *To Approve* and
loses the approver links, the validity dates and the state, with no error.

---

## 6. The Time Off Allocation

**TOF-060. Creation state.** A creation whose values name any state other than *To Approve* is
rejected:

> Incorrect state for new allocation

**TOF-061. A regular allocation carries a positive amount.** A database check named
`_duration_check` requires the day amount to be strictly greater than zero when the allocation
type is `regular`; an accrual allocation may legitimately start at zero:

> The duration must be greater than 0.

**TOF-062. The validity window is ordered.** Writing a non-empty validity end date that
precedes the validity start date is rejected:

> The Start Date of the Validity Period must be anterior to the End Date.

**TOF-063. An allocation may not be reduced below what has been consumed.** On any write that
touches the displayed day amount, the displayed hour amount or the state, the employee's
**non-provisional** excess for the type is measured before and after the write:

```formula
excess before = sum of the amounts of the excess entries that are not provisional, before the write
excess after  = sum of the amounts of the excess entries that are not provisional, after the write
```

When the excess has grown, and either the type forbids a negative balance or the new excess
exceeds the maximum excess amount, the write is rejected:

> You cannot reduce the duration below the duration of leaves already taken by the employee.

Only excess entries produced by *Approved* absences are counted, so a pending absence never
blocks a reduction.

**TOF-064. Approving one's own allocation.** Approving or refusing an allocation whose employee
is the acting user's own employee record is rejected when the type's allocation approval ladder
is not "None needed" and the actor is not an Administrator:

> Only a time off Administrator can approve/refuse their own requests.

**TOF-065. Allocation approval routing.** See `TOF-041` for the message raised when an
allocation may neither be validated nor moved to *Second Approval*.

**TOF-066. Allocation refusal precondition.** See `TOF-044` for the message raised when an
allocation is not in *To Approve*, *Second Approval* or *Approved*.

**TOF-067. Deleting an allocation.** Two independent guards apply, both suppressed when the
operation carries the allocation state-check suppression flag, which only the employee departure
procedure sets:

- an allocation whose state is neither *To Approve* nor *Refused* may not be deleted: *"You
  cannot delete an allocation request which is in `<state label>` state."*;
- an allocation of a type that requires an allocation and whose consumed amount is strictly
  greater than zero may not be deleted: *"You cannot delete an allocation request which has some
  validated leaves."*

**TOF-068. Re-initialisation of the accrual cursor.** Changing the validity start date, the
validity end date, the Accrual Plan or the employee of an accrual allocation that is **not yet
Approved** resets the cursor completely and replays the engine from the validity start date up
to the earlier of the validity end date and today, so that the user sees the amount the plan
would already have granted before saving. The reset values are in
[state-machines.md, section 4.4](state-machines.md#44-re-initialisation-of-the-cursor). An
allocation already *Approved* is never reset this way.

**TOF-069. Hour-based allocations follow the working schedule.** Whenever the working schedule
in force for an employee changes, the stored day amount of every hour-based allocation of that
employee is recomputed from its stored hour amount divided by the new hours per day. Without
this restatement the hours accrued under the previous schedule would be silently revalued.

---

## 7. Mandatory days and optional holidays

**TOF-070. Applicability of a Mandatory Day.** A Mandatory Day (`hr.leave.mandatory.day`)
applies to an employee over a date window when all of the following hold: its period intersects
the window; its company is one of the acting companies; its working schedule is empty or equals
the employee's working schedule; the employee has no job position, or the job position set is
empty, or the set contains the employee's job position; and either the employee has a department
and the department set is empty or contains that department or one of its ancestors, or the
employee has no department and the department set is empty. The full statement is in
[entities.md, section 8.1](entities.md#81-applicability-rule).

**TOF-071. Absence is forbidden on a Mandatory Day.** After the coverage check, and only for an
actor who does **not** hold the Officer group, a request whose period intersects a Mandatory Day
applicable to its employee is rejected:

> You are not allowed to request time off on a Mandatory Day

**TOF-072. Officers are not blocked.** An Officer may create absence for an employee on a
Mandatory Day. The per-request flag is still computed and still colours the calendar; only the
rejection of `TOF-071` is suppressed.

**TOF-073. The per-request flag is narrower than the applicability rule.** The flag the screens
read to show the warning additionally requires the Mandatory Day's working schedule to be empty
or to equal the **request's** working schedule, rather than the employee's current one, and,
when the request's type carries a company, requires the Mandatory Day to belong to that company.
A request can therefore be blocked by `TOF-071` while the screen shows no warning, and the
reverse cannot happen. This is recorded as a **compatibility finding**: a corrected behaviour
would evaluate one rule in both places; a compatible rebuild reproduces both, because the
warning is advisory and the rejection is authoritative.

**TOF-074. A Mandatory Day is ordered.** A database check named `_date_from_after_day_to`
requires the start date to be on or before the end date:

> The start date must be anterior than the end date.

**TOF-075. Requests limited to Optional Holidays.** With the Indian localization package
installed, a type may be flagged "Limited to optional holidays"
(`l10n_in_is_limited_to_optional_days`). A request of such a type whose requested period covers
at least one day that is not declared as an Optional Holiday
(`l10n.in.hr.leave.optional.holiday`) is rejected with:

> The following leaves are not on Optional Holidays:

followed by one line per offending request, each line consisting of a newline, a space, a
hyphen, a space and the request's display name. The comparison is made on the total length of
the intersection between the requested period and the union of the declared optional days,
rounded to two decimal places, against the total length of the requested period.

**TOF-076. Deleting an Optional Holiday.** Deleting an Optional Holiday that a request of a type
limited to optional holidays covers is rejected:

> You cannot delete an Optional Holiday that is linked to a leave request.

**TOF-077. Creating an Optional Holiday outside India.** Opening the creation form for an
Optional Holiday while the acting company's country is not India is refused:

> You must be logged in an Indian company to use this feature

---

## 8. Accrual plans and levels

**TOF-080. Milestone start consistency.** A database check named `_start_count_check` requires
the start count to be strictly positive when the milestone is reached "After", and exactly zero
when it is reached "At allocation creation":

> You can not start an accrual in the past.

Writing "At allocation creation" forces the start count to zero, and a start count of zero forces
the milestone to "At allocation creation".

**TOF-081. The rate is positive.** A database check named `_added_value_greater_than_zero`
requires the rate (`added_value`) to be strictly greater than zero:

> You must give a rate greater than 0 in accrual plan levels.

**TOF-082. A limited carry-over must carry something.** A database check named
`_valid_postpone_max_days_value` requires that the unused accruals are not carried over, or the
carry-over is not limited, or the maximum to carry over (`postpone_max_days`) is strictly greater
than zero:

> You cannot have a maximum quantity to carryover set to 0.

**TOF-083. A carried-over validity must last.** A database check named
`_valid_accrual_validity_value` requires the carried-over validity to be off or its count to be
strictly greater than zero:

> You cannot have an accrual validity time set to 0.

**TOF-084. A yearly cap must name an amount.** A database check named `_valid_yearly_cap_value`
requires the yearly cap to be off or the yearly maximum to be strictly greater than zero:

> You cannot have a cap on yearly accrued time without setting a maximum amount.

**TOF-085. A balance cap must name an amount.** Enabling the balance cap without a strictly
positive maximum is rejected:

> You cannot have a balance cap on accrued time set to 0.

**TOF-086. A weekly milestone needs a weekday.** A level whose frequency is "Weekly" with no
weekday chosen is rejected:

> Weekday must be selected to use the frequency weekly

**TOF-087. Twice-a-month day ordering.** A level whose frequency is "Twice a month" whose first
day is greater than or equal to its second day is rejected:

> The first day must be lower than the second day.

**TOF-088. Per Hour Worked is incompatible with granting at the start.** With the attendance
companion package installed, a level whose frequency is "Per Hour Worked" on a plan that grants
at the start of the accrual period is rejected:

> You can't base accrued time on hours worked, because time is accrued at the start of the period.

Correspondingly, switching a plan to grant at the start of the period silently converts every
"Per Hour Worked" level into an "Hourly" one.

**TOF-089. A used plan may not be deleted.** Deleting an Accrual Plan at which at least one
accrual allocation in a state other than *Refused* points is rejected:

> Some of the accrual plans you're trying to delete are linked to an existing allocation. Delete or cancel them first.

The check does not run while the capability package is being removed. Because an allocation has
no cancelled state, the practical condition is that any non-refused accrual allocation blocks the
deletion.

### 8.1 Derived settings, forced rather than refused

These situations produce a silent correction, not an error message.

| Situation | Forced value |
|---|---|
| The plan does not allow carry-over | every level's action with unused accruals becomes "Lost" |
| A level's action with unused accruals is "Lost" | its carry-over option becomes "Unlimited" and its carried-over validity is cleared |
| A level does not cap the running balance | its maximum becomes zero |
| The plan restricts a Time Off Type | every level's rate unit becomes `day` for a day-based or half-day-based type and `hour` for an hour-based type |
| The plan restricts no type | every level after the first copies the first level's rate unit; writing the unit on the first level writes the plan's value unit |
| The plan grants at the start of the period | the plan is forced not to be based on worked time, and a "Per Hour Worked" frequency becomes "Hourly" |
| A level's start count is zero | its milestone becomes "At allocation creation" |
| A level's milestone is "At allocation creation" | its start count becomes zero |
| A day-of-month selection exceeds the length of its month | it is clamped to the length of that month, using a leap year as the reference so that the twenty-ninth of February stays selectable |
| A plan is created with no name | it is named "Unnamed Plan" |

An unrecognised frequency raises, from either boundary function:

> Your frequency selection is not correct: please choose a frequency between theses options:Hourly, Daily, Weekly, Twice a month, Monthly, Twice a year and Yearly.

---

## 9. Access, visibility and company consistency

### 9.1 Groups and what they imply

| Group | Implies | Meaning |
|---|---|---|
| Internal User | — | Any employee. May file their own requests and read their own entitlement. |
| Time Off Responsible | Internal User | Someone who is the designated Approver of at least one employee. Granted and revoked automatically; never assigned by hand in practice. |
| Officer: Manage all requests | Time Off Responsible, Human Resources Officer | Reads and manages every request and allocation of the allowed companies. |
| Administrator | Officer: Manage all requests | Configures types, accrual plans, public holidays and mandatory days; may do anything. |

**TOF-090. Request visibility and writability.** Every internal user reads every request of
their own employee record. An Approver reads the requests of the employees they approve. An
Officer and an Administrator read every request of the allowed companies. Write access is
governed by the record rules of
[configuration.md, section 6.1](configuration.md#61-time-off-request), whose practical effect is:
an employee may write on their own request only while it is *To Approve*, *Refused* or
*Cancelled*; once it reaches *Second Approval* or *Approved* only an Officer, an Administrator or
the employee's Approver may change it; and an Officer may not write on their **own** *Approved*
request.

**TOF-091. Allocation visibility and writability.** Every internal user reads the allocations of
their own employee record and of the employees they approve. An Officer reads every allocation;
an Administrator reads and writes every allocation. The internal-user deletion rule names the
state `draft`, which the allocation state machine never takes, so a plain internal user can never
delete an allocation. This is recorded as a **compatibility finding**: the rule grants nothing; a
corrected behaviour would name *To Approve*, and a compatible rebuild keeps the rule inert so that
deletion stays an Officer power.

**TOF-092. Multi-company scoping.** A request is visible only when its company is among the
allowed companies. An allocation is visible when it has no employee or its employee's company is
allowed, **and** its type's company is allowed or empty. A type is visible when its company is
allowed, or its company is empty and its country is empty or is a country of one of the allowed
companies. An Accrual Plan, a Mandatory Day, the analysis projection and the calendar projection
are visible when their company is empty or allowed.

**TOF-093. Description masking.** Reading the description of a request returns the stored private
description (`private_name`) only when the reader is an Officer, or the login user of the
request's employee, or that employee's Approver; every other reader receives the five characters
`*****`. Writing the description writes the stored value only under the same test and is
otherwise silently ignored. Searching the description searches the stored column, restricted to
the reader's own requests unless the reader is an Officer.

**TOF-094. The Time Off Responsible group follows the approver link.** Naming a user as an
employee's Approver grants that user the Time Off Responsible group. Replacing or clearing the
approver revokes the group from the previous approver when no employee names them any more. The
same revocation sweep runs after every creation of login users. Archiving an employee empties the
approver link on that employee.

**TOF-095. Batch generation mode restriction.** A user who is not an Officer may use only the "By
Employee" mode of the multi-employee request dialog and of the multi-employee allocation dialog.
Any other mode is rejected:

> As Time Off Responsible, you can only use the allocation mode 'By Employee'.

**TOF-096. Batch generation and hour-based conflicts.** When the multi-employee request dialog
finds a conflicting request whose type is hour-based, the whole generation is rejected with:

> Some employees already have time off requests in hours that overlap with the selected period, `<the generation dialog's display name>` cannot automatically adjust or split hourly leaves during batch generation. Conflicting time off:

followed by one line per conflicting request, each of the shape *"- `<request display name>`"*.

**TOF-097. Approving or refusing from the overview calendar.** The two operations exposed by the
calendar projection are refused when the acting user is neither an Officer nor the employee's
Approver on a type whose request approval ladder is "By Employee's Approver" or "By Employee's
Approver and Time Off Officer":

> You are not allowed to approve this leave request.

> You are not allowed to refuse this leave request.

**TOF-098. Following a record that is already approved.** Adding a follower to, or mentioning a
user on, a request in state *Second Approval* or *Approved*, or an allocation in state *Approved*,
requires read access and is then performed with elevated rights, because the record rules would
otherwise forbid the write. This is what lets an employee keep discussing an absence after it has
been approved.

**TOF-099. A company's country is frozen while country-bound absence exists.** Changing the
country of a company is rejected when at least one request or one allocation exists whose employee
belongs to that company and whose type is bound to a country other than the new one:

> The company country cannot be changed while time off leaves or allocations with the country exist.

The guard is suppressed while the automated test suite runs.

---

## 10. Timesheet lines and payroll work entries

**TOF-100. Public-holiday timesheet lines are protected from deletion.** Deleting an Analytic Line
that carries a public-holiday link (`global_leave_id`) is rejected:

> You cannot delete timesheets that are linked to global time off.

**TOF-101. Absence timesheet lines are protected from deletion.** Deleting an Analytic Line that
carries an absence link (`holiday_id`) is rejected:

> You cannot delete timesheets that are linked to time off requests. Please cancel your time off request from the Time Off application instead.

When the actor is an Officer, or the absence belongs to the actor, the rejection additionally
offers a link opening the underlying request, labelled "View Time Off".

**TOF-102. Absence and public-holiday timesheet lines are read-only.** Writing on an Analytic Line
that carries a public-holiday link is rejected with *"Timesheets linked to public holidays cannot
be modified."*; writing on one that carries an absence link is rejected with *"You cannot modify
timesheets that are linked to time off requests. Please use the Time Off application to modify your
time off requests instead."* Both checks are bypassed for the domain's own elevated operations.

**TOF-103. No manual timesheet on a time off task.** Creating an Analytic Line whose task is a time
off task is rejected:

> You cannot create timesheets for a task that is linked to a time off type. Please use the Time Off application to request new time off instead.

A task counts as a time off task when at least one Analytic Line already links it to an absence or
to a public holiday, or when it is the acting company's configured time off task.

**TOF-104. A zero-duration request keeps no timesheet lines.** After any write on a request, every
request whose duration became zero has its timesheet lines unlinked and deleted.

**TOF-105. Cancelling a work entry refuses its absence.** Writing the cancelled state onto a work
entry that carries an absence link refuses that absence, unless the absence is already *Refused*.

**TOF-106. A validated work entry blocks cancellation.** Restated from `TOF-045`: the cancellation
permission of a request is forced to false when at least one validated work entry points at it.

**TOF-107. The long-term absence codes are exempt from the zero-duration rule.** Restated from
`TOF-042`: a type mapped to the work entry type codes `LEAVE110`, `LEAVE210` or `LEAVE280` may be
validated with a duration of zero days.

**TOF-108. Work entry type precedence over an absence interval.** For an interval of a working
schedule covered by absence, the work entry type is chosen in this order: a type flagged as
bypassing carried by the interval itself; then, among the Working Time Exclusions that fully
contain the interval, one whose absence's type maps to a bypassing code; then one with no absence
link, that is, a public holiday; then one with an absence link; and failing everything the generic
absence work entry type. A record with an absence link resolves its work entry type through its
Time Off Type; a record without one uses its own work entry type.

**TOF-109. French part-time gap filling.** With the French localization packages installed, a work
entry generation for an employee of a French company whose working schedule differs from the
company's working schedule additionally produces one work entry per company attendance interval,
carrying the absence's work entry type, for every date inside a validated absence whose end
instant was extended by the French duration rule (`TOF-131`) and that carries no work entry yet.
The generated entry is named *"`<work entry type name>`: `<employee name>`"* when the type is set
and *"`<employee name>`"* when it is not. For those same employees the step that marks absences
falling outside the working schedule is skipped entirely.

---

## 11. Extra hours convertible into time off

**TOF-110. An absence deducting extra hours needs enough of them.** With the attendance companion
package installed, after creating a request, after approving one, and after any write touching the
duration, the requested dates, the state, the employee or the type, every request of a type that
deducts extra hours (`overtime_deductible`) and requires no allocation is checked. When the
employee's unspent compensable extra hours would become negative, the operation is rejected with
*"You do not have enough extra hours to request this leave"* when the request belongs to the acting
user, and with *"The employee does not have enough extra hours to request this leave."* otherwise.
The arithmetic is in
[calculations.md, chapter 11](calculations.md#11-extra-hours-convertible-into-time-off).

**TOF-111. An allocation deducting extra hours needs enough of them.** After creating an allocation,
and after any write touching the amount or the type, every allocation of a type that deducts extra
hours is checked; when the unspent compensable extra hours would become negative the operation is
rejected:

> The employee does not have enough overtime hours to request this leave.

**TOF-112. Editing an allocation amount outside *To Approve*.** With the attendance companion
package installed, a user who is not an Officer writing the amount or the type of an allocation
whose state is not *To Approve* is rejected:

> Only an Officer or Administrator is allowed to edit the allocation duration in this status.

**TOF-113. The combined overtime rate.** When an overtime rule set combines its rates by summation
and at least one contributing rule is paid, the combined rate is one, plus the sum over the paid
rules that are not compensable as time off of their rate minus one, plus the sum over the paid
rules that are compensable as time off of their rate. An overtime line is marked compensable when
any contributing rule is compensable.

---

## 12. Public holidays and working time exclusions

**TOF-120. Public holidays may not overlap.** Two Working Time Exclusions **without a resource**,
belonging to the same company, whose periods intersect, and whose working schedules are compatible
— either record naming no schedule, or both naming the same schedule — are refused:

> Two public holidays cannot overlap each other for the same working hours.

**TOF-121. A working time exclusion is ordered.** The start instant must be earlier than the end
instant:

> The start date of the time off must be earlier than the end date.

**TOF-122. End-instant defaulting.** When the end instant is empty or is not strictly after the
start, it is set to the last second of the start's local day — twenty-three hours, fifty-nine
minutes and fifty-nine seconds — in the reading user's time zone, or in the acting company's
schedule's time zone when the reader has none.

**TOF-123. Time-zone reinterpretation at creation.** When a record is created with a working
schedule, with no resource, with both instants supplied, and the acting user's time zone differs
from the schedule's time zone, both instants are reinterpreted: read in the acting user's time
zone, restated in the schedule's time zone, converted back to coordinated universal time. A holiday
typed as "the whole of the twenty-fifth of December" is therefore stored as the whole of that day
**in the schedule's own time zone**.

**TOF-124. Company derivation.**

```formula
company = the company of the employee behind the absence link
company = the company of the working schedule , when there is no absence link
company = the acting company                  , when neither is available
```

**TOF-125. Public-holiday changes restate absences.** Creating, writing or deleting a resource-less
exclusion triggers the restatement procedure of
[workflows.md, chapter 9](workflows.md#9-create-change-or-delete-a-public-holiday) on every request
in a state other than *Refused* and *Cancelled* whose employee's company matches and whose period
strictly overlaps the changed period.

**TOF-126. The sick-absence exemption.** The notification *"Due to a change in global time offs,
`<number>` extra day(s) have been taken from your allocation. Please review this leave if you need
it to be changed."* is not sent for requests of the shipped Sick Time Off type, because absences of
that type are typically not funded by an allocation. The days are still taken; only the message is
suppressed.

**TOF-127. A restatement may refuse a request.** When the coverage check fails after a restatement,
the request is refused rather than left invalid, and the employee is notified with *"Due to a change
in global time offs, this leave no longer has the required amount of available allocation and has
been set to refused. Please review this leave."*

---

## 13. Rounding, dates and units

**TOF-130. Day-based rounding.** For a type whose request unit is `day`, the computed day count is
rounded **up** to the next whole number after the working schedule has produced it. A request
covering three working days and one public holiday therefore still costs three days, and a request
covering two and a quarter working days costs three days. When the rounding raises the figure, the
notice of `TOF-132` tells the requester.

**TOF-131. Half-day rounding on duration-based schedules.** For a type whose request unit is
`half_day`, when the working schedule expresses its attendance lines as durations rather than as
clock-hour ranges, the computed day count is rounded to the nearest multiple of one half.

**TOF-132. The rounding notice.** When the type is day-based, requires an allocation, and the
unrounded schedule duration is strictly smaller than the stored duration in days, the form shows:

> According to your working schedule you are expected to work `<unrounded days>` days in this period, but `<charged days>` days will be used because this leave `<type name>` can only be taken by days.

**TOF-133. Display rounding.** The duration text is the day figure rounded to two decimal places
with trailing zeros removed followed by the word "days"; for an hour-based type it is the hour
figure as hours and minutes separated by a colon, the minutes padded to two digits and rounded to
the nearest whole minute, a rounded value of sixty carrying into one more hour with zero minutes,
followed by the word "hours". Balance figures shown on screens are rounded to two decimal places.
The employee-aware display name of a type prints the provisionally remaining and the maximum
allowed amounts rounded to two decimal places without trailing zeros.

**TOF-134. Hour clamping.** The start hour of an hour-based request is clamped into the closed
interval from zero to 23.99; the end hour is clamped into the closed interval from zero to
twenty-four.

**TOF-135. Day-of-month clipping.** Every day-of-month selector paired with a month selector — the
two twice-a-year days, the yearly day and the plan's carry-over day — is clipped to the number of
days that month has **in a leap year**, which keeps the twenty-ninth of February selectable and
turns a thirty-first into a twenty-ninth or a thirtieth where appropriate.

**TOF-136. Time-zone resolution order.** Converting a requested date and clock hour into an instant
uses the first non-empty value among: the time zone of the employee's resource, the time zone of the
request's working schedule, the time zone of the acting company's working schedule, the acting
user's time zone, and coordinated universal time.

**TOF-137. Return-date resolution.** The date an employee is back is the date of the first working
interval that begins after the end of the approved absence covering the present instant. The search
window widens through seven, thirty, ninety, one hundred and eighty, three hundred and sixty-five
and seven hundred and thirty days until an interval is found; when none is found the return date
stays empty.

**TOF-138. Accrual unit conversion.** A grant expressed in hours is converted into days by dividing
by the employee's hours per day on the allocation's validity start date. A balance cap, a yearly cap
and a carry-over maximum expressed in hours are converted the same way, using the employee's hours
per day on the relevant accrual date. An employee with no working schedule at all is treated as
having twenty-four hours per day; where neither an employee nor a schedule is available, eight hours
per day is used.

**TOF-139. Zero-length fragments are dropped.** Whenever a request is split — by the multi-employee
generation, by an Employee Version change or by an employee departure — a resulting fragment whose
absolute start is not strictly before its absolute end is discarded rather than created.

---

## 14. Automatic cancellation of invalid absence

**TOF-140. The daily invalid-absence rule.** A daily scheduled process protects accrual-driven
entitlement from being over-consumed by absences placed further in the future than the accrual can
support.

1. Consider the window from today at zero hours to the day thirty-one days from today at
   twenty-three hours, fifty-nine minutes, fifty-nine seconds and 999999 microseconds.
2. Select the requests whose absolute start falls in that window and whose state is *To Approve*,
   *Second Approval* or *Approved*, ordered by absolute start **descending**.
3. Select the accrual allocations of those employees, for those types, in state *Approved*, whose
   validity overlaps the window.
4. Keep only the requests whose type appears among those allocations, re-sorted by absolute start
   descending.
5. For each remaining request, compute the balance data of the employee for the type at the
   request's start date:
   - when the maximum allowed is zero, cancel the request;
   - otherwise let the allowed excess be the type's maximum excess amount when the type allows a
     negative balance and zero otherwise; when the total provisional excess is at or below that
     allowance the request is kept, and otherwise it is cancelled.

The cancellation reason posted is:

> the accruated amount is insufficient for that duration.

It is posted as an internal **note**, and the responsible approvers are **not** notified.

**TOF-141. Cancelling the furthest absence first.** Processing the requests from the latest start
date backwards is deliberate: cancelling the furthest request first frees entitlement for the nearer
ones, so the earliest absences survive.

---

## 15. Edge cases and their resolutions

**TOF-150. A request placed entirely on non-working time.** Created successfully with a duration of
zero days and zero hours; it cannot be validated, by `TOF-042`.

**TOF-151. A public holiday declared over an existing request.** Handled by the restatement
procedure of [workflows.md, chapter 9](workflows.md#9-create-change-or-delete-a-public-holiday),
which recomputes the duration, informs the employee of the change, and refuses the request when the
new duration no longer fits the entitlement.

**TOF-152. A working schedule changed on an employee.** Every **future** request of the employee
whose schedule differs from the new one is rewritten to the new schedule, its absolute dates are
recomputed — except for an hour-based request, whose clock hours the user chose explicitly — and the
Working Time Exclusions of the approved ones are rebuilt. When the recomputed durations no longer
fit, the write is refused by `TOF-029`.

**TOF-153. An Employee Version created or amended across an existing request.** Handled by
[workflows.md, section 10.1](workflows.md#101-creating-a-version). When the split produces fragments
whose durations no longer fit, the operation is refused with the second message of `TOF-029`.

**TOF-154. A department or approver change on an employee.** Changing the employee's hierarchical
parent sets the Approver to the parent's login user, but only for employees whose current approver
is the previous parent's login user or who have no approver at all. Changing the department or the
parent rewrites the department on every request of the employee that is *To Approve* or starts in
the future, and on every allocation of the employee that is *To Approve*, where the manager link is
rewritten as well.

**TOF-155. An archived employee.** Archiving empties the Approver link on the employee. Requests and
allocations remain; because both reference the employee through a link marked as restricting
deletion, the employee record itself cannot be deleted while any exist.

**TOF-156. Multi-company request creation.** The company of a request follows its employee. A user
acting for one company who creates a request for an employee of another lands the record in the
employee's company, where the multi-company rule may immediately hide it from the creator; the
platform offers to switch to that company.

**TOF-157. An allocation with no end date placed before one with an end date.** The consumption order
of `TOF-054` means an unlimited allocation is always consumed last, whatever its start date. An
employee holding both a year-limited grant and an unlimited one therefore burns the expiring one
first.

**TOF-158. An accrual allocation whose plan has no levels.** Silently skipped by the accrual engine;
the balance never moves.

**TOF-159. An accrual allocation whose plan has not started.** When the target date falls before the
first level's transition date and the cursor has never run, the allocation is skipped entirely and
the next call date stays empty, so a later run retries.

**TOF-160. An accrual level whose balance cap has already been exceeded.** The grant amount becomes
negative, because the cap formula subtracts the current balance from the capped total, and the
balance therefore **decreases**. This is the behaviour and a rebuild must not clamp the grant at
zero. It is recorded as a **compatibility finding**: a corrected behaviour would clamp the grant at
zero and leave a manually raised balance untouched.

**TOF-161. A carry-over cut-off that coincides with the reference date.** The cut-off function
compares strictly, so a reference date equal to the cut-off keeps that year's cut-off rather than
moving to the next. The closest-expiry computation compensates by adding one year when the two
coincide, because entitlement accrued **on** the cut-off belongs to the following carry-over period.

**TOF-162. A half-day request on a duration-based schedule.** The proportional day figure is rounded
to the nearest half day by `TOF-131`. On any other schedule the figure is left as computed, which on
an irregular schedule can be a value such as one and a half or one.

**TOF-163. An hour-based request whose window contains no working time.** Duration zero hours,
duration text `0:00 hours`. It can be created and cannot be validated.

**TOF-164. A fully flexible employee.** Such an employee has no working schedule at all. Duration is
measured on the wall clock, hours per day is taken as twenty-four, and public holidays are still
subtracted, by whole local days.

**TOF-165. Requests that start after the balance reference date.** Excluded from the ordinary
charging pass by `TOF-058` and handled by the deferred set, so that a request placed in December is
not charged against an accrual balance that will only exist in December. The shortfall is reported as
the exceeding duration, a negative number, rather than as an excess entry.

**TOF-166. A request of a type that requires no allocation.** The whole duration is charged to a
pseudo-allocation with no identity: the provisionally taken figure grows, both remaining figures are
set to zero, and an *Approved* request additionally grows the taken figure. No excess is ever
recorded for such a type.

---

## 16. Index of rule identifiers

| Rule | Title | Chapter |
|---|---|---|
| TOF-001 … TOF-012 | The twelve invariants | [1](#1-invariants) |
| TOF-020 | An employee is mandatory | [2](#2-the-request-period-and-its-duration) |
| TOF-021 | The resolved period is ordered | 2 |
| TOF-022 | The requested period is ordered | 2 |
| TOF-023 | The duration is not negative | 2 |
| TOF-024 | Overlap | 2 |
| TOF-025 | A request may not span Employee Versions with different working schedules | 2 |
| TOF-026 | Payroll period locking | 2 |
| TOF-027 | The resolved period is frozen after the first approval | 2 |
| TOF-028 | Modifying a request that has already begun | 2 |
| TOF-029 | A working schedule change may not invalidate existing absences | 2 |
| TOF-030 | The negative cap carries an amount | [3](#3-the-time-off-type) |
| TOF-031 | Request stacking is reserved for worked time | 3 |
| TOF-032 | Worked time is always eligible for the accrual rate | 3 |
| TOF-033 | The public-holiday inclusion flag is frozen while it would move balances | 3 |
| TOF-034 | The allocation requirement is frozen once the type has been used | 3 |
| TOF-035 | Country derivation | 3 |
| TOF-036 | Company and country are frozen once the type is used | 3 |
| TOF-037 | Ordering and default selection | 3 |
| TOF-038 | Deleting a type | 3 |
| TOF-039 | Type usability | 3 |
| TOF-040 | Every state write passes the guard ladder | [5](#5-deleting-refusing-cancelling-and-reopening) |
| TOF-041 | Approve routing | 5 |
| TOF-042 | Zero-duration approval | 5 |
| TOF-043 | Validation state check | 5 |
| TOF-044 | Refusal precondition | 5 |
| TOF-045 | Cancellation precondition | 5 |
| TOF-046 | The double-approval rule | 5 |
| TOF-047 | Reopening precondition | 5 |
| TOF-048 | Deleting a request | 5 |
| TOF-049 | Duplicating a request | 5 |
| TOF-050 | An allocation must exist | [4](#4-entitlement-coverage) |
| TOF-051 | Coverage when a negative balance is allowed | 4 |
| TOF-052 | Coverage when no negative balance is allowed | 4 |
| TOF-053 | When the coverage check runs | 4 |
| TOF-054 | Consumption ordering | 4 |
| TOF-055 | Unit of the balance arithmetic | 4 |
| TOF-056 | Excess recording | 4 |
| TOF-057 | Pending consumption | 4 |
| TOF-058 | Future absences funded by accrual are deferred | 4 |
| TOF-059 | An allocation that is not approved funds nothing | 4 |
| TOF-060 | Creation state | [6](#6-the-time-off-allocation) |
| TOF-061 | A regular allocation carries a positive amount | 6 |
| TOF-062 | The validity window is ordered | 6 |
| TOF-063 | An allocation may not be reduced below what has been consumed | 6 |
| TOF-064 | Approving one's own allocation | 6 |
| TOF-065 | Allocation approval routing | 6 |
| TOF-066 | Allocation refusal precondition | 6 |
| TOF-067 | Deleting an allocation | 6 |
| TOF-068 | Re-initialisation of the accrual cursor | 6 |
| TOF-069 | Hour-based allocations follow the working schedule | 6 |
| TOF-070 | Applicability of a Mandatory Day | [7](#7-mandatory-days-and-optional-holidays) |
| TOF-071 | Absence is forbidden on a Mandatory Day | 7 |
| TOF-072 | Officers are not blocked | 7 |
| TOF-073 | The per-request flag is narrower than the applicability rule | 7 |
| TOF-074 | A Mandatory Day is ordered | 7 |
| TOF-075 | Requests limited to Optional Holidays | 7 |
| TOF-076 | Deleting an Optional Holiday | 7 |
| TOF-077 | Creating an Optional Holiday outside India | 7 |
| TOF-080 | Milestone start consistency | [8](#8-accrual-plans-and-levels) |
| TOF-081 | The rate is positive | 8 |
| TOF-082 | A limited carry-over must carry something | 8 |
| TOF-083 | A carried-over validity must last | 8 |
| TOF-084 | A yearly cap must name an amount | 8 |
| TOF-085 | A balance cap must name an amount | 8 |
| TOF-086 | A weekly milestone needs a weekday | 8 |
| TOF-087 | Twice-a-month day ordering | 8 |
| TOF-088 | Per Hour Worked is incompatible with granting at the start | 8 |
| TOF-089 | A used plan may not be deleted | 8 |
| TOF-090 | Request visibility and writability | [9](#9-access-visibility-and-company-consistency) |
| TOF-091 | Allocation visibility and writability | 9 |
| TOF-092 | Multi-company scoping | 9 |
| TOF-093 | Description masking | 9 |
| TOF-094 | The Time Off Responsible group follows the approver link | 9 |
| TOF-095 | Batch generation mode restriction | 9 |
| TOF-096 | Batch generation and hour-based conflicts | 9 |
| TOF-097 | Approving or refusing from the overview calendar | 9 |
| TOF-098 | Following a record that is already approved | 9 |
| TOF-099 | A company's country is frozen while country-bound absence exists | 9 |
| TOF-100 | Public-holiday timesheet lines are protected from deletion | [10](#10-timesheet-lines-and-payroll-work-entries) |
| TOF-101 | Absence timesheet lines are protected from deletion | 10 |
| TOF-102 | Absence and public-holiday timesheet lines are read-only | 10 |
| TOF-103 | No manual timesheet on a time off task | 10 |
| TOF-104 | A zero-duration request keeps no timesheet lines | 10 |
| TOF-105 | Cancelling a work entry refuses its absence | 10 |
| TOF-106 | A validated work entry blocks cancellation | 10 |
| TOF-107 | The long-term absence codes are exempt from the zero-duration rule | 10 |
| TOF-108 | Work entry type precedence over an absence interval | 10 |
| TOF-109 | French part-time gap filling | 10 |
| TOF-110 | An absence deducting extra hours needs enough of them | [11](#11-extra-hours-convertible-into-time-off) |
| TOF-111 | An allocation deducting extra hours needs enough of them | 11 |
| TOF-112 | Editing an allocation amount outside To Approve | 11 |
| TOF-113 | The combined overtime rate | 11 |
| TOF-120 | Public holidays may not overlap | [12](#12-public-holidays-and-working-time-exclusions) |
| TOF-121 | A working time exclusion is ordered | 12 |
| TOF-122 | End-instant defaulting | 12 |
| TOF-123 | Time-zone reinterpretation at creation | 12 |
| TOF-124 | Company derivation | 12 |
| TOF-125 | Public-holiday changes restate absences | 12 |
| TOF-126 | The sick-absence exemption | 12 |
| TOF-127 | A restatement may refuse a request | 12 |
| TOF-130 | Day-based rounding | [13](#13-rounding-dates-and-units) |
| TOF-131 | Half-day rounding on duration-based schedules | 13 |
| TOF-132 | The rounding notice | 13 |
| TOF-133 | Display rounding | 13 |
| TOF-134 | Hour clamping | 13 |
| TOF-135 | Day-of-month clipping | 13 |
| TOF-136 | Time-zone resolution order | 13 |
| TOF-137 | Return-date resolution | 13 |
| TOF-138 | Accrual unit conversion | 13 |
| TOF-139 | Zero-length fragments are dropped | 13 |
| TOF-140 | The daily invalid-absence rule | [14](#14-automatic-cancellation-of-invalid-absence) |
| TOF-141 | Cancelling the furthest absence first | 14 |
| TOF-150 … TOF-166 | The seventeen edge cases | [15](#15-edge-cases-and-their-resolutions) |

---

## 17. Mapping of former rule identifiers

The two source drafts of this folder numbered their rules differently. One draft used chapter and
section numbers together with the invariant labels I1 to I10; the other used a prefix of its own
with a three-digit sequence. Both are mapped onto the scheme of this file below, so that no citation
written against either draft is lost.

### 17.1 From the section-numbered draft

| Former reference | New rule |
|---|---|
| Invariants I1 … I10 | TOF-001 … TOF-010 |
| 2.1 Creation requires an employee | TOF-020 |
| 2.2 Date ordering | TOF-021, TOF-022 |
| 2.3 Non-negative duration | TOF-023 |
| 2.4 Overlap | TOF-024 |
| 2.5 Deleting a request | TOF-048 |
| 2.6 Modifying a request | TOF-027, TOF-028 |
| 2.7 Duplication | TOF-049 |
| 2.8 The employment-term constraint | TOF-025 |
| 2.9 Zero-duration approval | TOF-042, TOF-107 |
| 2.10 Entitlement validity | TOF-050, TOF-051, TOF-052, TOF-053 |
| 2.11 Mandatory days | TOF-071 |
| 2.12 Overtime-deductible types | TOF-110, TOF-111, TOF-112 |
| 2.13 Work entry locking | TOF-026, TOF-106 |
| 3.1 Changing the allocation requirement | TOF-034 |
| 3.2 The negative cap | TOF-030 |
| 3.3 Requests on top | TOF-031 |
| 3.4 Accrual eligibility of worked time | TOF-032 |
| 3.5 Freezing the public-holiday inclusion flag | TOF-033 |
| 3.6 Deleting a type | TOF-038 |
| 4.1 Creation state | TOF-060 |
| 4.2 Amount | TOF-061 |
| 4.3 Validity window | TOF-062 |
| 4.4 Reducing an allocation below what has been consumed | TOF-063 |
| 4.5 Deleting an allocation | TOF-067 |
| 4.6 Approving one's own allocation | TOF-064 |
| 4.7 Duplication | TOF-049 |
| 5 Accrual plan and level rules | TOF-080 … TOF-089 |
| 5.1 Derived constraints | [section 8.1](#81-derived-settings-forced-rather-than-refused) |
| 6.1 Overlapping public holidays | TOF-120 |
| 6.2 Date ordering | TOF-121 |
| 6.3 End date defaulting | TOF-122 |
| 6.4 Company derivation | TOF-124 |
| 7.1 Groups | [section 9.1](#91-groups-and-what-they-imply) |
| 7.2 Automatic membership of the Responsible group | TOF-094 |
| 7.3 Who may act on a request | TOF-040, and [state-machines.md](state-machines.md#24-the-same-map-as-tables) |
| 7.4 The double-validation rule | TOF-046 |
| 7.5 The description privacy rule | TOF-093 |
| 7.6 The multi-employee wizards | TOF-095 |
| 7.7 Approving from the calendar report | TOF-097 |
| 7.8 Subscription to the discussion thread | TOF-098 |
| 8.1 … 8.3 Record rules | TOF-090, TOF-091, TOF-092, and [configuration.md, chapter 6](configuration.md#6-record-rules) |
| 9 Access rights matrix | [configuration.md, chapter 5](configuration.md#5-model-access-rights) |
| 10 Locking rules | TOF-026, TOF-027, TOF-028, TOF-045, TOF-063, TOF-034, TOF-033, TOF-099 |
| 11 Automatic cancellation of invalid absence | TOF-140, TOF-141 |
| 12 Mandatory days | TOF-070, TOF-071, TOF-072, TOF-073 |
| 13.1 … 13.16 Edge cases | TOF-150 … TOF-165 |

### 17.2 From the sequence-numbered draft

| Former identifier | New rule |
|---|---|
| 001 Negative cap amount | TOF-030 |
| 002 Request stacking reserved for worked time | TOF-031 |
| 003 Worked time always eligible for the accrual rate | TOF-032 |
| 004 Public holiday inclusion frozen | TOF-033 |
| 005 Allocation requirement frozen | TOF-034 |
| 006 No request across versions with different schedules | TOF-025 |
| 007 Company and country of a used type frozen | TOF-036 |
| 008 Country derivation | TOF-035 |
| 009 Type ordering and default selection | TOF-037 |
| 010 Resolved period ordering | TOF-021 |
| 011 Requested period ordering | TOF-022 |
| 012 Non-negative duration | TOF-023 |
| 013 An employee is mandatory | TOF-020 |
| 014 Overlap | TOF-024 |
| 015 Resolved period frozen after the first approval | TOF-027 |
| 016 Mandatory Working Day | TOF-071 |
| 017 Double approval rights | TOF-046 |
| 018 Modifying a request that already began | TOF-028 |
| 019 Modifying a cancelled request | TOF-040 |
| 020 An allocation must exist | TOF-050 |
| 021 Coverage with a negative balance allowed | TOF-051 |
| 022 Coverage without a negative balance | TOF-052 |
| 023 When the coverage check runs | TOF-053 |
| 024 Allocation consumption ordering | TOF-054 |
| 025 Unit of the balance arithmetic | TOF-055 |
| 026 Excess recording | TOF-056 |
| 027 Type usability | TOF-039 |
| 028 Pending consumption | TOF-057 |
| 029 Future absences funded by accrual are deferred | TOF-058 |
| 030 The approval matrix | TOF-040 |
| 031 No repeated action | TOF-040 |
| 032 Second Approval only with two approvals | TOF-040 |
| 033 A cancelled request is immutable | TOF-040 |
| 034 Duplication | TOF-049 |
| 035 Deletion by a user who is not an Officer | TOF-048 |
| 036 Deletion by an Officer who is not an Administrator | TOF-048 |
| 037 Approve routing | TOF-041 |
| 038 A request with no working time cannot be validated | TOF-042 |
| 039 Refusal precondition | TOF-044 |
| 040 Cancellation precondition | TOF-045 |
| 041 Reopening precondition | TOF-047 |
| 042 An actor may not approve their own allocation | TOF-064 |
| 043 Allocation approval precondition | TOF-041, TOF-065 |
| 044 Allocation refusal precondition | TOF-044, TOF-066 |
| 045 Allocation deletion guards | TOF-067 |
| 046 Validity ordering | TOF-062 |
| 047 Regular allocations carry a positive amount | TOF-061 |
| 048 Creation state | TOF-060 |
| 049 An allocation may not be reduced below what has been consumed | TOF-063 |
| 050 Public holidays may not overlap | TOF-120 |
| 051 Public holiday time zone reinterpretation | TOF-123 |
| 052 Resource Time Off company derivation | TOF-124 |
| 053 Public holiday changes restate absences | TOF-125 |
| 054 The sick absence exemption | TOF-126 |
| 055 Restatement may refuse a request | TOF-127 |
| 056 Mandatory Working Day applicability | TOF-070 |
| 057 Officers bypass Mandatory Working Days | TOF-072 |
| 060 Milestone start consistency | TOF-080 |
| 061 Positive grant | TOF-081 |
| 062 A limited carry-over must carry something | TOF-082 |
| 063 A carry-over validity must last | TOF-083 |
| 064 A yearly cap must name an amount | TOF-084 |
| 065 A weekly milestone needs a weekday | TOF-086 |
| 066 Twice a month day ordering | TOF-087 |
| 067 A balance cap must name an amount | TOF-085 |
| 068 Per hour worked incompatible with granting at the start | TOF-088 |
| 069 A used plan may not be deleted | TOF-089 |
| 070 Public holiday timesheet lines protected from deletion | TOF-100 |
| 071 Absence timesheet lines protected from deletion | TOF-101 |
| 072 Absence and public holiday timesheet lines read-only | TOF-102 |
| 073 No manual timesheet on a time off task | TOF-103 |
| 074 A zero duration request keeps no timesheet lines | TOF-104 |
| 080 Cancelling a work entry refuses its absence | TOF-105 |
| 081 A validated work entry blocks cancellation | TOF-106 |
| 082 Long term absence codes exempt from the zero duration rule | TOF-107 |
| 083 Work entry type precedence over an absence interval | TOF-108 |
| 084 An absence deducting extra hours needs enough of them | TOF-110 |
| 085 An allocation deducting extra hours needs enough of them | TOF-111 |
| 086 Editing an allocation amount outside To Approve | TOF-112 |
| 087 Combined overtime rate | TOF-113 |
| 090 Request visibility | TOF-090 |
| 091 Allocation visibility | TOF-091 |
| 092 Multi company scoping | TOF-092 |
| 093 Description masking | TOF-093 |
| 094 Company country change guard | TOF-099 |
| 095 The Time Off Responsible group follows the approver field | TOF-094 |
| 096 Schedule change guard | TOF-029 |
| 097 Batch generation mode restriction | TOF-095 |
| 098 Batch generation and hourly conflicts | TOF-096 |
| 099 Following an approved record | TOF-098 |
| 100 Day based rounding | TOF-130 |
| 101 Half day rounding on duration based schedules | TOF-131 |
| 102 Display rounding | TOF-133 |
| 103 Hour clamping | TOF-134 |
| 104 Day of month clipping | TOF-135 |
| 105 Time zone resolution order | TOF-136 |
| 106 Return date resolution | TOF-137 |
| 107 Accrual unit conversion | TOF-138 |
| 108 Fully flexible employees | TOF-164, and [calculations.md, section 4.2](calculations.md#42-branch-a--flexible-employees) |
| 109 Zero length fragments are dropped | TOF-139 |

---

## 18. Reconciliation notes

1. **Two numbering schemes.** One draft numbered its rules by chapter and section, the other with a
   three-digit sequence. Both are replaced by the single `TOF-` scheme of this file and mapped in
   [chapter 17](#17-mapping-of-former-rule-identifiers). The numbers cited by
   [workflows.md](workflows.md) are the numbers of this file.
2. **Where the guard messages live.** One draft listed the per-transition refusal messages here, the
   other in a dedicated state-machine document. The ordered ladder is normative in
   [state-machines.md](state-machines.md#29-the-guard-ladder-and-its-messages) and is restated once,
   without an order-dependent duplicate, as `TOF-040`.
3. **The record-rule tables.** One draft carried the full record-rule domains here and the other in
   its configuration document. The tables now live in
   [configuration.md, chapter 6](configuration.md#6-record-rules); `TOF-090` to `TOF-092` state their
   effect and point at them, and no row was dropped in the move.
4. **The coverage-check trigger list.** One draft said the check runs on a write touching the
   requested start date, the absolute dates, the type, the employee or the state; the other added
   that it also runs during a public-holiday restatement, where a failure refuses the request. Both
   statements are true and are merged in `TOF-053` and `TOF-127`.
5. **The allocation deletion state guard.** One draft said the guard is bypassed by the departure
   procedure alone, the other did not mention the suppression flag. The source confirms a single
   caller, so `TOF-067` keeps the narrower statement.
6. **Country-specific rules.** One draft omitted the Indian and French rules on the grounds that they
   belong to a localization domain. The packages that add them are inside the scope of this folder,
   so `TOF-075`, `TOF-076`, `TOF-077` and `TOF-109` specify them here, with the arithmetic in
   [calculations.md, chapter 15](calculations.md#15-country-specific-duration-rules).
7. **The narrower mandatory-day flag.** Only one draft recorded that the flag the screens read is
   narrower than the rule that blocks the request. It is retained as `TOF-073` and marked as a
   compatibility finding.
8. **The inert allocation deletion rule.** Both drafts observed that the internal-user deletion rule
   on an allocation names a state the machine never reaches. It is kept as a compatibility finding
   under `TOF-091` rather than silently corrected.
