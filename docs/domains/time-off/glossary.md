# Time Off — Glossary

Every term of the domain, defined. Terms owned by another domain are marked with the name of that
domain and are defined here only as far as this domain uses them. Where a term has a stored
identifier, the identifier is given in code font beside it.

---

## A

**Absence.** Two meanings, always distinguished by context. As a **setting**, the value `leave`
labelled "Absence" of a Time Off Type's kind of time off: the time does not count as worked time, is
subtracted from working intervals, and makes the employee appear absent. As a **word**, an informal
name for a Time Off Request.

**Absence ledger.** The read-only projection `hr.leave.attendance.report` that carries one row per
employee per day with the expected hours, the worked hours, the approved absence hours and their
difference. Added by the attendance companion package.

**Absolute layer.** The pair of instants `date_from` and `date_to`, stored in coordinated universal
time, computed from the request layer and never written directly. See *Request layer*.

**Accrual allocation.** A Time Off Allocation whose allocation type is `accrual`: its amount is grown
by an Accrual Plan rather than typed once.

**Accrual bonus.** The amount an accrual allocation will have granted between today and a chosen
future evaluation date, obtained by simulating the engine on a discarded copy.

**Accrual cursor.** The five fields — last call, actual last call, next call, already accrued, last
executed carry-over date — that together record how far the accrual engine has advanced an
allocation.

**Accrual date.** A date on which an accrual period closes and the grant lands.

**Accrual Plan** (`hr.leave.accrual.plan`). The named configuration that describes how an accrual
allocation grows: when the grant lands, whether it is prorated on worked time, where the carry-over
cut-off sits, how the plan moves from one milestone to the next, and which Time Off Type it is
restricted to.

**Accrual Plan Level** (`hr.leave.accrual.level`). One milestone of an Accrual Plan. See *Milestone*.

**Accrual period.** The interval between two consecutive boundaries of the milestone in force.

**Actual last call** (`actual_lastcall`). The date of the last iteration the accrual engine performed
on an allocation, whether or not that iteration granted anything. It differs from the last call on
carry-over cut-offs, on level transition dates and on carried-over expiry dates.

**Allocated amount.** The gross amount an allocation carries, before any consumption. Stored in days
and shown in hours for an hour-based Time Off Type.

**Allocation.** Short for Time Off Allocation.

**Allocation ladder** (`allocation_validation_type`). The approval path a Time Off Allocation
follows: None needed, By Time Off Officer, By Employee's Approver, or By Employee's Approver and Time
Off Officer.

**Analytic Line** (`account.analytic.line`). *Analytic accounting domain.* The record that carries a
quantity of time against an analytic account, a project and a task. This domain generates one per
working day of an approved absence when the timesheet companion package is installed.

**Approved** (`validate`). The terminal favourable state of both a Time Off Request and a Time Off
Allocation.

**Approver.** Short for Time Off Approver.

**Attendance** (`hr.attendance`). *Attendances and working time domain.* A recorded arrival and
departure of an employee. Used here only through the overtime lines and the absence ledger.

**Available balance.** The remaining amount of an allocation, or of a Time Off Type, after approved
and pending consumption. Called the provisionally remaining balance where the distinction from the
approved-only figure matters.

## B

**Balance cap.** Another name for the *running cap*, used on the milestone dialog. See *Running cap*.

**Both approvals.** The informal name of the "By Employee's Approver and Time Off Officer" ladder,
the only ladder that uses the *Second Approval* state.

**Branch A, Branch B, Branch C.** The three mutually exclusive paths of the duration algorithm: the
wall-clock branch for flexible employees, the count-of-scheduled-days branch for a day-based type,
and the proportional branch for everything else. See
[calculations.md, section 4.3](calculations.md#43-why-the-three-day-figures-differ).

**Bridging day.** *Indian localization.* A non-working day enclosed by, or adjacent to, two absences
of a bridging-enabled type, counted inside the duration.

## C

**Cancelled** (`cancel`). The terminal state a Time Off Request reaches when it is withdrawn, always
with the option of a reason. A cancelled request is immutable and its materialisation is reversed. A
Time Off Allocation has no cancelled state.

**Carried-over pool** (`expiring_carryover_days`). The balance recorded on a carry-over cut-off,
which is the amount subject to the carried-over validity window.

**Carry-over cut-off, carry-over anchor** (`carryover_date`). The yearly date on which the carry-over
policy of a milestone is applied: the first of January, the anniversary of the allocation's validity
start date, or a custom month and day.

**Carry-over limit** (`postpone_max_days`). The largest amount a milestone lets survive a carry-over
cut-off.

**Carry-over validity** (`accrual_validity`). The window, counted in days or months from the cut-off,
after which the carried-over pool expires.

**Chargeable duration.** The part of an absence that falls inside one allocation's validity window,
measured against the employee's working schedule when the intersection is narrower than the absence.

**Company working schedule.** *Contacts and organizations, with attendances and working time.* The
schedule attached to a company, used as the fallback whenever an employee's own schedule cannot be
resolved.

**Consumption order.** The order in which absences charge the allocations of one employee and one
type: allocations carrying an end date first, ascending by that end date; then accrual allocations
with no end date; then regular allocations with no end date. Rule `TOF-054`.

**Coverage check.** The validation that a Time Off Request is funded by an approved allocation. Rules
`TOF-050` to `TOF-053`.

## D

**Deferred absence.** A Time Off Request starting after the evaluation date and funded by an accrual
allocation. It is not charged at that date; the deferred set is compared as a block against the
balance the accrual will have reached. Rule `TOF-058`.

**Department** (`hr.department`). *Human resources core domain.* The organisational unit of an
employee. Used here for scoping, for counters and for Mandatory Day restrictions.

**Distinct-preserving.** A property of an interval set: two intervals that merely touch are kept
separate instead of being merged. Attendance and work sets carry it; exclusion sets do not.

**Duration text** (`duration_display`). The human-readable rendering of a duration: days with two
decimal places and trailing zeros removed, or hours and minutes separated by a colon for an
hour-based type.

## E

**Employee Version** (`hr.version`). *Human resources core domain.* One period of an employee's
employment terms, carrying the working schedule and the contract dates. A request may not span
versions carrying different schedules.

**Evaluation date.** The date for which a balance is computed. It defaults to today; the dashboard
lets the reader move it.

**Exceeding duration** (`exceeding_duration`). The shortfall a set of deferred absences will still
have after the accrual has run. Zero or negative.

**Excess entry** (`excess_days`). A record, keyed by the end date of an absence, of the part of that
absence no allocation could fund, together with a flag saying whether the absence is still pending.

**Extra hours.** *Attendances and working time domain.* Overtime an employee has accumulated. When
the overtime rule marks it compensable as time off, it may be converted into absence of a type
flagged as deducting extra hours.

## F

**First approval.** The transition from *To Approve* to *Second Approval*, which exists only under
the two-approval ladder. The button that performs it is labelled Approve, while the button that
performs the validation is labelled Validate.

**Flexible schedule, fully flexible employee.** *Attendances and working time domain.* A working
schedule declaring average hours instead of fixed attendance lines, and an employee carrying no
working schedule at all. Durations for such employees are measured on the wall clock rather than on
schedule intervals.

## G

**Grant.** The amount a milestone adds to an allocation when a period opens or closes.

**Grant unit** (`added_value_type`). Days or Hours, the unit in which a milestone expresses its rate.
Forced by the Time Off Type when the plan is restricted to one.

**Gross balance** (`number_of_days` on an allocation). The running accrued amount of an allocation,
including the part already consumed by absence. Every cap is applied to the gross balance minus the
consumed part.

## H

**Half day** (`half_day`). A request unit that lets an absence start or end at the midpoint of a
working day. The midpoint comes from the schedule's morning and afternoon attendance lines, or from
the midpoint of a full-day line when the schedule carries no separate halves.

**Hours per day.** *Attendances and working time domain.* The conventional working hours of one day on
a working schedule. Used to convert between the day amount and the hour amount of an allocation and
of a grant. Twenty-four for an employee with no working schedule at all, and eight where neither an
employee nor a schedule is available.

## I

**Internal project.** *Timesheets domain.* The company project that receives absence timesheet lines.

**Interval.** A start instant, an end instant and a payload. Every schedule computation of this domain
works on ordered, disjoint sets of intervals.

## L

**Ladder.** See *Request ladder* and *Allocation ladder*.

**Last call** (`lastcall`). The date of the last accrual period boundary on which a grant landed.

**Level.** See *Milestone*.

**Level transition date.** The date on which a milestone starts: the allocation's validity start date
plus the milestone's offset.

**Level transition mode** (`transition_mode`). Whether a milestone change takes effect on its
transition date, "Immediately", or only once the running period has closed, "After this accrual's
period".

## M

**Mandatory Day** (`hr.leave.mandatory.day`), called a Mandatory Working Day on some screens. A date range on which absence is forbidden to anybody
who is not an Officer, optionally restricted to a working schedule, to departments and to job
positions.

**Materialisation.** The set of records written when a Time Off Request reaches *Approved*: the
Working Time Exclusion, the Calendar Event, the timesheet lines, the work entries and the approval
message.

**Maximum allowed** (`max_leaves`). The total entitlement granted to an employee for one type at one
date, the projected accrual gain included.

**Maximum excess amount** (`max_allowed_negative`). The largest negative balance a type allows,
expressed in that type's request unit.

**Milestone.** One stage of an Accrual Plan, stored as an Accrual Plan Level: it starts a fixed offset
after the allocation's validity start date, grants a fixed amount at a fixed frequency, and carries
its own caps and carry-over policy.

## N

**Negative balance** (`allows_negative`). The setting that lets an employee consume more entitlement
than they hold, down to the maximum excess amount.

**Next call** (`nextcall`). The next date on which the accrual engine must act on an allocation. Empty
means the plan has never run on it.

## O

**Officer.** Short for Time Off Officer, the holder of the access group labelled "Officer: Manage all
requests".

**Optional Holiday** (`l10n.in.hr.leave.optional.holiday`). *Indian localization.* A single calendar
day a company declares as eligible for a flexible absence.

**Overlap warning** (`dashboard_warning_message`). The message produced when a Time Off Request would
sit on top of another request of the same employee that is neither refused nor cancelled. It is both
a displayed warning and the text of the blocking validation.

## P

**Pending consumption.** The part of a balance consumed by requests in state *To Approve* or *Second
Approval*. Counted by every provisional figure and by none of the plain figures.

**Period proration.** The factor that reduces the grant of a partial accrual period to the share of
the period actually covered.

**Provisionally remaining** (`virtual_remaining_leaves`). The maximum allowed minus approved
consumption minus pending consumption. This is the figure every screen, every type-usability test and
every coverage guard reads.

**Public holiday.** A Working Time Exclusion with no resource, either company-wide or attached to one
working schedule. Public holidays shorten absence durations unless the Time Off Type says otherwise.

## R

**Rate** (`added_value`). How much a milestone grants each period, expressed in the grant unit.

**Refused** (`refuse`). The terminal unfavourable state of both a Time Off Request and a Time Off
Allocation.

**Regular allocation** (`regular`). A Time Off Allocation whose amount is typed once and never grows.

**Remaining** (`remaining_leaves`). The maximum allowed minus approved consumption alone.

**Request.** Short for Time Off Request.

**Request ladder** (`leave_validation_type`). The approval path a Time Off Request follows: None
needed, By Time Off Officer, By Employee's Approver, or By Employee's Approver and Time Off Officer.

**Request layer.** The fields a person fills in: the two requested dates, the two day periods and the
two clock hours. The absolute layer is derived from it.

**Request unit** (`request_unit`). Day, Half-Day or Hours: the granularity in which a type may be
requested, which also decides the rounding of the duration and the unit of the balance arithmetic.

**Requested dates** (`request_date_from`, `request_date_to`). The plain calendar dates the requester
types, before they are resolved into instants.

**Resolved instants.** See *Absolute layer*.

**Rounding notice** (`leave_type_increases_duration`). The advisory text shown when a day-based type
rounds the duration up. Rule `TOF-132`.

**Running cap** (`cap_accrued_time`, `maximum_leave`). The milestone setting that stops the available
balance of an allocation from rising above a fixed amount. It caps the available balance, not the
gross balance, so an employee who takes absence keeps accruing.

## S

**Second Approval** (`validate1`). The intermediate state between *To Approve* and *Approved*,
reachable only under the two-approval ladder.

**Supporting document.** An attachment on a Time Off Request. A type may declare that it expects them.

## T

**Taken** (`leaves_taken`). The part of a balance consumed by requests in state *Approved*.

**Time Off Allocation** (`hr.leave.allocation`). One entitlement grant of one type to one employee,
valid between two dates, either a fixed amount or driven by an Accrual Plan.

**Time Off Analysis** (`hr.leave.report`). The read-only projection that unions allocations, carrying
positive amounts, and requests, carrying negated amounts.

**Time Off Approver** (`leave_manager_id`). The login user named on an employee record as responsible
for approving that employee's absences. Being named grants the Time Off Responsible group
automatically.

**Time Off Balance by Employee and Type** (`hr.leave.employee.type.report`). The read-only projection
that pairs, per employee and per type, the remaining balance of each allocation with the absences
taken and planned.

**Time Off Calendar Report** (`hr.leave.report.calendar`). The read-only projection of one row per
non-cancelled request, flattened with the employee's time zone, job position and company, that feeds
the company-wide overview calendar.

**Time Off Officer.** The holder of the access group "Officer: Manage all requests". Reads and
approves everything of the allowed companies and manages public holidays.

**Time Off Request** (`hr.leave`). One absence of one employee over one period, with its state, its
duration and its approvers.

**Time Off Responsible.** The access group granted automatically to every Time Off Approver. It gives
read access to the records of the employees they approve and the right to use the two multi-employee
dialogs in "By Employee" mode.

**Time Off Type** (`hr.leave.type`). The catalogue entry that defines how one kind of absence is
requested, approved, counted, paid and displayed.

**To Approve** (`confirm`). The initial state of both a Time Off Request and a Time Off Allocation.
There is no draft state.

## U

**Unpaid** (`unpaid`). A flag on a Time Off Type read by payroll and by the accrual proration that
excludes unpaid absence from worked time.

**Unspent compensable extra hours.** The extra hours an employee has accumulated, minus the absences
and allocations of extra-hour-deducting types already booked against them.

**Unusual day.** A day on which an employee is not expected to work, shaded by the calendar widgets.

## V

**Validation.** The transition to *Approved*, performed by the Validate button. Distinct from the
first approval, which only moves a record to *Second Approval*.

**Validity period** (`date_from`, `date_to` on an allocation). The window during which an entitlement
may be consumed. An empty end date means no expiry.

**Virtual figures.** See *Provisionally remaining* and *Pending consumption*. The word *virtual*
survives in the stored identifiers `virtual_remaining_leaves` and `virtual_leaves_taken`; in prose
this folder says *provisional*.

## W

**Work entry** (`hr.work.entry`). *Work entries domain.* One quantity of time carrying a payroll code,
produced from an approved absence and consumed by payroll.

**Work entry type** (`hr.work.entry.type`). *Work entries domain.* The payroll code carried by a work
entry. A Time Off Type may map to one.

**Worked Time** (`other`). The value of a Time Off Type's kind of time off that makes the time count
as worked time. Such a type never makes the employee look absent, is always eligible for the accrual
rate, is the only kind that may allow requests on top of another absence, and produces no timesheet
line.

**Worked time proration.** The factor that reduces a grant to the share of the accrual period the
employee actually worked, counting absences flagged eligible for the accrual rate as worked.

**Working schedule** (`resource.calendar`). *Attendances and working time domain.* The pattern of
attendance lines, the time zone and the hours per day against which every duration in this domain is
measured.

**Working Time Exclusion** (`resource.calendar.leaves`), labelled "Resource Time Off Detail" on the
screens. *Attendances and working time domain.* The record that tells every availability computation
that a resource is unavailable over a period. Written by this domain at validation and removed on
refusal, cancellation, reopening and deletion. A record without a resource is a public holiday.

## Y

**Yearly cap** (`cap_accrued_time_yearly`, `maximum_leave_yearly`). The milestone setting that limits
the total granted between two carry-over cut-offs. The counter is reset to zero at every cut-off.

---

## Reconciliation notes

1. **Only one draft carried a glossary.** Its entries are kept and extended with the vocabulary the
   other draft introduced: the request and absolute layers, the three duration branches, the
   distinct-preserving interval property, the chargeable duration, the gross balance, the accrual
   cursor, the rounding notice and the maximum excess amount.
2. **Virtual against provisional.** One draft used the word *virtual* throughout, the other
   *provisional*. The stored identifiers carry *virtual* and are reproduced; the prose of this folder
   says *provisional*, and the entry **Virtual figures** records the equivalence once.
