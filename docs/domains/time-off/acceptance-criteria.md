# Time Off — Acceptance Criteria

Numbered, independently verifiable Given / When / Then scenarios that a rebuild of this domain must
satisfy. They cover every configuration guard, every state transition, every formula, every workflow
and every access rule of this folder, each with concrete records, concrete inputs and exact results.

Unless a scenario says otherwise, the fixture is the following.

- **Company** "Main", country Belgium, time zone two hours ahead of coordinated universal time in
  summer and one hour ahead in winter, working schedule "Standard 40 hours": Monday to Friday, a
  morning attendance line from 08:00 to 12:00 and an afternoon attendance line from 13:00 to 17:00,
  eight hours a day, one-week pattern, clock-hour ranges rather than durations, not flexible.
- **Employees**, all of Main and all on "Standard 40 hours": Alice, whose Time Off Approver is Mark;
  Bob, whose Time Off Approver is Mark; Mark, who holds only the Internal User group and is therefore
  a Time Off Responsible by virtue of being an approver; Olivia, who holds the Officer group; Adele,
  who holds the Administrator group.
- **Time Off Types**: "Paid Time Off", request unit Day, allocation required, request ladder By Time
  Off Officer, allocation ladder By Time Off Officer, negative balance not allowed; "Sick Time Off",
  request unit Day, no allocation required, request ladder By Time Off Officer; "Unpaid Hours",
  request unit Hours, no allocation required, request ladder None needed; "Family Days", request
  unit Half-Day, allocation required, request ladder By Employee's Approver and Time Off Officer.

---

## 1. Configuring a Time Off Type

**AC-001.** Given a new Time Off Type with the negative cap enabled and a maximum excess amount of
zero, when it is saved, then the save is rejected with "The maximum excess amount should be greater
than 0. If you want to set 0, disable the negative cap instead." (`TOF-030`)

**AC-002.** Given a Time Off Type counted as Absence, when request stacking is enabled on it, then
the save is rejected with "You cannot allow requests on top of leaves of type 'Absence'."
(`TOF-031`)

**AC-003.** Given a Time Off Type counted as Worked Time, when it is created, then its accrual
eligibility flag is true; and when the flag is cleared, the save is rejected with "leaves of type
'Worked Time' should be always eligible for accrual rate." (`TOF-032`)

**AC-004.** Given a Time Off Type counted as Absence, when it is created, then its accrual
eligibility flag is false, and an Administrator may set it to true without error. (`TOF-032`)

**AC-005.** Given Alice holds an approved request of Paid Time Off from the twenty-fourth to the
twenty-sixth of December 2024 and a public holiday exists on the twenty-fifth, when an Administrator
changes the public-holiday inclusion flag of Paid Time Off, then the save is rejected with "You
cannot modify the 'Public Holiday Included' setting since one or more leaves for that                         time off type are overlapping with public holidays, meaning that the balance of those employees would be affected by this change." (`TOF-033`)

**AC-006.** Given at least one request of Paid Time Off exists, when an Administrator clears the
allocation requirement on that type, then the save is rejected with "The allocation requirement of a
time off type cannot be changed once leaves of that type have been taken. You should create a new
time off type instead." (`TOF-034`)

**AC-007.** Given a Time Off Type with no company, when an Administrator sets the company to Main
whose country is Belgium, then the type's country becomes Belgium; and when the company is cleared
again, the country stays Belgium. (`TOF-035`)

**AC-008.** Given a Time Off Type that at least one request references, when its form is opened,
then the company and the country fields are read-only. (`TOF-036`)

**AC-009.** Given the six shipped types with sequences 1, 2, 3, 4, 4 and 5, when the catalogue is
listed with no employee in the calling context, then the order is Paid Time Off, Sick Time Off,
Unpaid, Compensatory Days, Extra Time Off, Extra Hours, the two types of sequence 4 in an
unspecified relative order. (`TOF-037`)

**AC-010.** Given Alice holds an approved allocation of three days of Paid Time Off and the calling
context names Alice, when the employee-aware display name of Paid Time Off is read, then it reads
"Paid Time Off (3 remaining out of 3 days)".

**AC-011.** Given a Time Off Type named "Paid Time Off", when it is duplicated, then the copy is
named "Paid Time Off (copy)" and every Accrual Plan restricted to the original is duplicated with
it.

**AC-012.** Given a Time Off Type referenced by one request, when it is deleted, then the deletion is
refused by the referential restriction; and when it is archived instead, then it disappears from
every selection list while the existing request stays readable. (`TOF-038`)

---

## 2. Resolving the period and computing the duration

**AC-013.** Given Paid Time Off, request unit Day, and a request from Monday the thirteenth to
Wednesday the fifteenth of May 2024, when it is saved, then the absolute start is the thirteenth of
May at 06:00 and the absolute end the fifteenth of May at 15:00 coordinated universal time, the
duration is three days and twenty-four hours, and the duration text is "3 days".

**AC-014.** Given Family Days, request unit Half-Day, and a request from Wednesday the eighth of May
2024 Morning to Friday the tenth of May 2024 Morning, with a public holiday covering the whole of
Thursday the ninth on the same schedule and the type excluding public holidays, when it is saved,
then the absolute start is the eighth of May at 06:00 and the absolute end the tenth of May at 10:00
coordinated universal time, the duration is 1.5 days and twelve hours, and the duration text is
"1.5 days".

**AC-015.** Given the same request on a type that includes public holidays in the duration, when it
is saved, then the duration is 2.5 days and twenty hours.

**AC-016.** Given the same period on a day-based type, when it is saved, then the absolute end is the
tenth of May at 15:00 coordinated universal time, the duration is two days and sixteen hours, and the
duration text is "2 days".

**AC-017.** Given Unpaid Hours, request unit Hours, and a request on Tuesday the fourteenth of May
2024 from nine to eleven thirty, when it is saved, then the duration is 2.5 hours and 0.3125 days,
and the duration text is "2:30 hours".

**AC-018.** Given a half-day request on a single day whose start half and end half are both Morning,
when it is saved, then the resolved instants are 08:00 and 12:00 local and the duration is half a
day.

**AC-019.** Given a half-day request on a single day whose start half and end half are both
Afternoon, when it is saved, then the resolved instants are 13:00 and 17:00 local and the duration is
half a day.

**AC-020.** Given a half-day request on a single day whose start half is Morning and end half is
Afternoon, when it is saved, then the resolved instants are 08:00 and 17:00 local and the duration is
one day.

**AC-021.** Given a working schedule whose Monday and Tuesday carry a single full-day attendance line
from ten to eighteen and no other line, and a half-day request on Monday the twenty-first of April
2025 from Morning to Morning, when it is saved, then the duration is half a day, because the full-day
group is split at its midpoint of fourteen.

**AC-022.** Given a working schedule whose attendance lines are expressed as durations of 3.36 hours
for each morning and each afternoon of Monday to Friday, and a half-day request from Monday the
twentieth to Friday the twenty-fourth of April 2026, Morning to Afternoon, when it is saved, then the
duration is exactly five days, the half-day rounding to the nearest half applying. (`TOF-131`)

**AC-023.** Given a day-based request covering two working days and one non-working day, when it is
saved, then the duration is two days and the rounding notice is empty, the unrounded figure equalling
the rounded one.

**AC-024.** Given a day-based type that requires an allocation and a request whose unrounded schedule
duration is 1.5 days over two calendar days, when it is saved, then the stored duration is two days
and the rounding notice reads "According to your working schedule you are expected to work 1.5 days
in this period, but 2.0 days will be used because this leave `<type name>` can only be taken by
days." (`TOF-130`, `TOF-132`)

**AC-025.** Given an employee with a fully flexible working schedule and a half-day-based type, and a
request from the eighth to the tenth of May 2024 Morning to Morning with a whole-day public holiday
on the ninth, when it is saved, then the duration is 1.5 days and twenty-six hours.

**AC-026.** Given a request whose employee is empty and whose working schedule is the company
schedule, when the duration is computed for the period from eight to twelve of a working Monday,
then the hours are four and the days are 4 ÷ 8 = 0.5.

**AC-027.** Given an hour-based request, when the start hour is typed as minus three, then it is
clamped to zero; when the end hour is typed as thirty, then it is clamped to twenty-four; when the
start hour is typed as twenty-five, then it is clamped to 23.99. (`TOF-134`)

**AC-028.** Given an hour-based request whose hours were left at the values the schedule proposed for
the fourteenth of May 2024, when the requested dates are moved to the twenty-first of May 2024, then
the hours are refreshed from the schedule for the new dates.

**AC-029.** Given an hour-based request whose hours the user typed as nine to eleven thirty, which
differ from the schedule proposal, when the requested dates are moved, then the typed hours are
preserved.

**AC-030.** Given a request whose absolute start falls after its absolute end, when it is saved, then
the save is rejected with "The start date must be before or equal to the end date." (`TOF-021`)

**AC-031.** Given a request whose requested start date falls after its requested end date, when it is
saved, then the save is rejected with "The request start date must be before or equal to the request
end date." (`TOF-022`)

**AC-032.** Given a creation whose values carry no employee, when it is executed, then it is rejected
with "There is no employee set on the time off. Please make sure you're logged in the correct
company." (`TOF-020`)

**AC-033.** Given an employee whose time zone is nine hours ahead of coordinated universal time while
the reader is two hours ahead, when an hour-based request of that employee is opened, then the form
shows the notice "The employee has a different timezone than yours! Here dates and times are
displayed in the employee's work timezone" followed by the time zone name.

**AC-034.** Given a request placed entirely on a Saturday and a Sunday, when it is saved, then it is
created with a duration of zero days and zero hours; and when an Officer validates it, then the
validation is rejected with "The following employees are not supposed to work during that period:"
followed by the employee name. (`TOF-042`, `TOF-150`)

---

## 3. Overlap

**AC-035.** Given Alice holds a request from the third to the seventh of June 2024 in state *To
Approve*, when Alice creates a second request of any type that does not allow requests on top, from
the fifth to the sixth of June, then the creation is rejected with a message beginning "You've
already booked time off which overlaps with this period:" and containing one line for the existing
request with its dates and the state label "To Approve". (`TOF-024`)

**AC-036.** Given Alice holds a request from the third to the seventh of June 2024 in state
*Refused*, when Alice creates a request from the fifth to the sixth of June, then the creation
succeeds, refused requests never conflicting.

**AC-037.** Given Alice holds a request ending on the seventh of June 2024 at 15:00 coordinated
universal time, when Alice creates a request starting on the seventh of June at 15:00 coordinated
universal time, then the creation succeeds, touching boundaries not overlapping.

**AC-038.** Given a Time Off Type counted as Worked Time with request stacking enabled, when a
request of that type is created over an existing absence, then no warning is produced and the
creation succeeds.

**AC-039.** Given Olivia, an Officer, creates a request for Bob over a period Bob already has a
request in, when it is saved, then the rejection message begins "An employee already booked time off
which overlaps with this period:" and names Bob.

**AC-040.** Given a refused request of Alice overlapping a period, when an approver opens the
calendar block of another request over the same period, then no blocking warning is shown, and the
calendar still displays the refused request struck through.

---

## 4. The approval ladders

**AC-041.** Given a Paid Time Off request of Alice in state *To Approve* under the By Time Off
Officer ladder, when Alice presses Approve, then the operation is rejected with "Only a Time Off
Officer/Manager can validate a leave." (`TOF-040`)

**AC-042.** Given the same request, when Mark, who is Alice's Approver but not an Officer, presses
Approve, then the operation is rejected with "Only a Time Off Officer/Manager can validate a
leave.", the By Time Off Officer ladder granting the Approver nothing.

**AC-043.** Given the same request, when Olivia, an Officer, presses Validate, then the state becomes
*Approved*, the first approver is Olivia's employee record, the second approver stays empty, a
Working Time Exclusion is created, and the employee receives the message "Your Paid Time Off planned
on `<absolute start in the request's time zone>` has been accepted".

**AC-044.** Given a request whose ladder is By Employee's Approver and whose employee is Alice, when
Mark presses Approve, then the state becomes *Approved* and the first approver is Mark's employee
record.

**AC-045.** Given a Family Days request of Alice in state *To Approve* under the two-approval ladder,
when Mark presses Approve, then the state becomes *Second Approval*, the first approver is Mark's
employee record, the Time Off Approval activity is marked done, and one Time Off Second Approve
activity is created for each notified officer of the type.

**AC-046.** Given that request in state *Second Approval*, when Mark presses Validate, then the
operation is rejected because Mark is not an Officer; and when Olivia presses Validate, then the
state becomes *Approved* and the second approver is Olivia's employee record. (`TOF-046`)

**AC-047.** Given a request of Olivia herself under the By Time Off Officer ladder, when Olivia
presses Validate, then the state becomes *Approved*, an Officer being allowed to validate their own
request.

**AC-048.** Given a request in state *Approved*, when Olivia presses Validate again, then the
operation is rejected with "You can't do the same action twice." (`TOF-040`)

**AC-049.** Given a Paid Time Off request, whose ladder is not the two-approval one, when any actor
tries to move it to *Second Approval*, then the operation is rejected with "Not possible state. State
Approve is only used for leave needed 2 approvals". (`TOF-040`)

**AC-050.** Given a request in state *Cancelled*, when Adele, an Administrator, tries to move it to
*Approved*, then the operation is rejected with "A cancelled leave cannot be modified." (`TOF-040`)

**AC-051.** Given a request in state *Refused*, when Olivia presses Validate, then the state becomes
*Approved*, an Officer being able to revive a refused request.

**AC-052.** Given a request in state *Refused* under the By Employee's Approver ladder, when Mark
presses Validate, then the state becomes *Approved*.

**AC-053.** Given a request in state *To Approve* under the By Time Off Officer ladder, when Mark
presses Refuse, then the operation is rejected with "You can't refuse a leave with validation by Time
Off Officer."

**AC-054.** Given a selection of two requests, one of which the actor may neither validate nor move
to *Second Approval*, when Approve is pressed on the selection, then the whole operation is rejected
with "You cannot approve this leave." and neither request changes state. (`TOF-041`)

**AC-055.** Given a non-Officer who is not the Approver of the employee concerned, when that user
tries to move a two-approval request to *Second Approval*, then the operation is rejected with "You
cannot first approve a time off for `<employee name>`, because you are not his time off manager".
(`TOF-046`)

---

## 5. Automatic approval, refusal, cancellation, reopening and deletion

**AC-056.** Given the Unpaid Hours type with the None needed ladder, when Alice creates a request of
that type, then the request is immediately *Approved*, the responsible approvers are subscribed, the
comment "The time off has been automatically approved" is posted, and a Working Time Exclusion
exists.

**AC-057.** Given an approved Paid Time Off request of Alice carrying a Calendar Event, when Olivia
refuses it, then the state becomes *Refused*, the second approver is Olivia's employee record, the
Calendar Event is archived, the Working Time Exclusion is removed, the approval activities are
removed, and Alice receives "Your Paid Time Off planned on `<absolute start>` has been refused".

**AC-058.** Given a Family Days request in state *Second Approval*, when Olivia refuses it, then the
**first** approver, not the second, becomes Olivia's employee record, and Alice's Approver Mark
receives a separate notification with the subject "Refused Time Off" and the body "`<request display
name>` has been refused."

**AC-059.** Given a request in state *Cancelled*, when Refuse is pressed, then the operation is
rejected with "Time off request must be confirmed or validated in order to refuse it." (`TOF-044`)

**AC-060.** Given an approved request of Alice starting tomorrow, when Alice opens the cancellation
dialog, types the reason "Plans changed" and confirms, then the state becomes *Cancelled*, an
internal note reading "The time off request has been cancelled for the following reason:" followed by
"Plans changed" is posted, the responsible approvers receive the subject "Cancelled Time Off", the
Working Time Exclusion is removed, the Calendar Event is archived, and a success notification reads
"Your time off has been cancelled."

**AC-061.** Given an approved request of Alice that started last week, when Alice tries to cancel it,
then the operation is rejected with "This time off cannot be cancelled."; and when Olivia, an
Officer, cancels the same request, then the cancellation succeeds. (`TOF-045`)

**AC-062.** Given an approved request of Bob, when Alice tries to cancel it, then the operation is
rejected with "This time off cannot be cancelled."

**AC-063.** Given a cancelled request, when Alice writes any field on it, then the write is rejected
with "Only a manager can modify a canceled leave." (`TOF-040`)

**AC-064.** Given an approved request, when Olivia presses Back to Approval, then the state becomes
*To Approve*, the Working Time Exclusion is removed, the Calendar Event is archived, and a new Time
Off Approval activity is created. (`TOF-047`)

**AC-065.** Given a request of Alice in state *To Approve* starting next week, when Alice deletes it,
then it is deleted.

**AC-066.** Given a request of Alice in state *To Approve* that started last week, when Alice deletes
it, then the deletion is rejected with "You can't delete a time off request that is in the past."
(`TOF-048`)

**AC-067.** Given a request of Alice in state *Approved*, when Alice deletes it, then the deletion is
rejected with "Oops! Approved Time-Off requests can only be deleted by Administrators."

**AC-068.** Given a request in state *Refused*, when Olivia, an Officer who is not an Administrator,
deletes it, then the deletion is rejected with "Oops! Refused Time-Off requests can only be deleted
by Administrators."; and when Adele, an Administrator, deletes it, then it is deleted.

**AC-069.** Given a request in state *To Approve*, when it is duplicated, then the duplication is
rejected with "A time off cannot be duplicated."; and given a request in state *Cancelled*, when it
is duplicated, then the copy is created. (`TOF-049`)

**AC-070.** Given a request of Alice that started yesterday and is in state *Approved*, when Bob, an
internal user who is neither an Officer nor Alice's Approver, writes the description on it, then the
write is rejected with "You must have manager rights to modify/validate a time off that already
begun". (`TOF-028`)

**AC-071.** Given an approved request of Alice, when Olivia changes its absolute dates, then the
write is rejected with "This modification is not allowed in the current state." (`TOF-027`)

---

## 6. Balance and coverage

**AC-072.** Given Alice holds no allocation of Paid Time Off, when she creates a request of that
type, then the creation is rejected with the two-line message "You do not have any allocation for
this time off type." followed by "Please request an allocation before submitting your time off
request." (`TOF-050`)

**AC-073.** Given Alice holds one approved allocation of two days of Paid Time Off valid from the
first of January 2024 with no end date, when she creates a request of three days, then the creation
is rejected with "Alice does not have a valid allocation for the leave type Paid Time Off to cover
that request." (`TOF-052`)

**AC-074.** Given the same allocation, when she creates a request of two days, then the creation
succeeds and the available balance becomes zero.

**AC-075.** Given Alice holds allocation A of ten days valid from the first of January to the
thirtieth of June 2024 and allocation B of fifteen days valid from the first of April 2024 with no
end date, both approved, and an approved request of five days from the eleventh to the fifteenth of
March and a request of three days in state *To Approve* from the sixth to the eighth of May, when the
balance is read at the first of May 2024, then allocation A shows allocated ten, taken five,
provisionally taken eight, remaining five and provisionally remaining two; allocation B shows
allocated fifteen, taken zero, provisionally taken zero, remaining fifteen and provisionally
remaining fifteen; and the aggregate shows allocated twenty-five, taken five, provisionally taken
eight, remaining twenty, provisionally remaining seventeen, requested three and approved five.

**AC-076.** Given the same fixture with the second request lasting eight days instead of three, when
the balance is read, then allocation A is exhausted, allocation B carries the remaining three days,
and allocation B's provisionally remaining balance becomes twelve.

**AC-077.** Given the same fixture with the second request lasting thirty days, when it is created,
then the creation is rejected with "Alice does not have a valid allocation for the leave type Paid
Time Off to cover that request."

**AC-078.** Given the same fixture with the second request of three days, when the closest expiry is
read at the first of May 2024, then the expiry date is the thirtieth of June 2024 and the expiring
amount is two days.

**AC-079.** Given Alice holds one approved allocation of four days valid from the first to the fifth
of June 2024 and one approved request from the third to the seventh of June of five working days,
when the balance is read, then the allocation is charged three days, its remaining balance is one,
and an excess entry of two days is recorded under the seventh of June. (`TOF-054`, `TOF-056`)

**AC-080.** Given a Time Off Type that requires no allocation, when Alice takes four days of it, then
the balance map records those four days under the empty key with both remaining figures at zero, and
no excess is ever recorded. (`TOF-166`)

**AC-081.** Given Alice already has an excess of two days recorded under the seventh of June 2024
from an earlier request, when she edits the description of an unrelated request, then no coverage
error is raised, the excess map being unchanged by the edit. (`TOF-052`)

**AC-082.** Given Alice holds an allocation valid from the first of January to the thirty-first of
December 2024 and a second valid from the first of January 2025, when the type selector is opened
with the requested dates in January 2025, then the type is offered, the usability test using the
requested dates. (`TOF-039`)

**AC-083.** Given Alice's only allocation of Paid Time Off expired on the thirty-first of December
2023, when she opens a new request dated in 2024, then Paid Time Off is not offered in the type
selector.

**AC-084.** Given Alice's only allocation of Paid Time Off is fully consumed, when she opens the
request form, then Paid Time Off is not offered, the form narrowing the list to the types that allow
a negative balance or still show a strictly positive provisionally remaining balance.

---

## 7. The negative balance

**AC-085.** Given a Time Off Type "Limited with negative" allowing a negative balance up to five
days, an approved allocation of one day valid for 2022 and an approved allocation of five days valid
from the first of January 2023 with no end date, when Alice creates a five-day request in October
2022, then the creation succeeds and the 2022 balance is minus four. (`TOF-051`)

**AC-086.** Given the same fixture, when Alice creates a five-day request in October 2023, then the
creation succeeds and the 2023 balance is minus four.

**AC-087.** Given the same fixture and the October 2023 request already recorded, when Alice creates
a second five-day request in October 2023, then the creation is rejected, the balance reaching minus
nine while only minus five is allowed.

**AC-088.** Given the same fixture and a one-day request bringing the balance to minus five, when an
Administrator extends that request by one more day, then the write is rejected, the balance reaching
minus six.

**AC-089.** Given a type that allows a negative balance and an employee holding no allocation at all,
when a request is created, then the rejection is the no-allocation message of AC-072, not the
coverage message. (`TOF-050`)

**AC-090.** Given a type allowing a negative balance of up to five days and an employee whose excess
is two days, when the dashboard tile is read, then the provisionally remaining balance shows minus
two and the total excess shows two; and given the same excess on a type that forbids a negative
balance, then the provisionally remaining balance shows zero while the total excess still shows two.

---

## 8. Time Off Allocations

**AC-091.** Given Alice, when she requests an allocation of five days of a type whose allocation
ladder is By Time Off Officer and which allows employee requests, then the allocation is created in
state *To Approve*, one Allocation Approval activity is created for each notified officer, and Alice,
the login user of Alice's hierarchical parent and Alice's Approver are subscribed.

**AC-092.** Given that allocation, when Alice presses Approve on it, then the operation is rejected
with "Only a time off Administrator can approve/refuse their own requests." (`TOF-064`)

**AC-093.** Given that allocation, when Olivia presses Validate, then the state becomes *Approved*
and the first approver is Olivia's employee record.

**AC-094.** Given an allocation whose ladder is the two-approval one, when Mark approves it, then the
state becomes *Second Approval* and the first approver is Mark; and when Olivia then validates it,
then the state becomes *Approved*, the second approver is Olivia and the first approver stays Mark.

**AC-095.** Given an allocation in state *Second Approval* whose first approver was never recorded,
when an Officer validates it, then both the first and the second approver become that Officer's
employee record.

**AC-096.** Given an allocation whose ladder is None needed, when Alice requests it, then it is
*Approved* immediately at the end of the creation.

**AC-097.** Given a creation whose values name the state *Approved*, when it is executed, then it is
rejected with "Incorrect state for new allocation". (`TOF-060`)

**AC-098.** Given an approved allocation of five days of which three have been consumed by approved
absence, when its amount is reduced to two days, then the write is rejected with "You cannot reduce
the duration below the duration of leaves already taken by the employee." (`TOF-063`)

**AC-099.** Given the same allocation, when its amount is reduced to four days, then the write
succeeds, the consumed amount still being covered.

**AC-100.** Given an approved allocation whose consumed amount is strictly positive, when it is
deleted, then the deletion is rejected with "You cannot delete an allocation request which has some
validated leaves." (`TOF-067`)

**AC-101.** Given an approved allocation with no consumption, when it is deleted, then the deletion
is rejected with "You cannot delete an allocation request which is in Approved state."

**AC-102.** Given an allocation in state *To Approve* with no consumption, when it is deleted, then
it is deleted.

**AC-103.** Given a validity start date of the first of May 2024 and a validity end date of the first
of April 2024, when the allocation is saved, then it is rejected with "The Start Date of the Validity
Period must be anterior to the End Date." (`TOF-062`)

**AC-104.** Given a Regular Allocation whose amount is zero, when it is saved, then it is rejected
with "The duration must be greater than 0."; and given an Accrual Allocation whose amount is zero,
when it is saved, then it is accepted. (`TOF-061`)

**AC-105.** Given an employee on an eight-hour day and an hour-based Time Off Type, when an
allocation of forty hours is recorded, then the stored day amount is five, the duration text reads
"40 hours" and the generated description reads "`<type name>` (40.0 hour(s))".

**AC-106.** Given the same allocation and the employee moving to a six-hour working schedule, when
the Employee Version is written, then the stored day amount becomes 6.666667 and the hour amount
stays forty. (`TOF-069`)

**AC-107.** Given an allocation whose description the user typed as "Seniority bonus", when the
amount is changed, then the description is not regenerated; and when the description is cleared, then
the generated title returns.

**AC-108.** Given an allocation duplicated by a user, when the copy is created, then its state is *To
Approve*, its approvers are empty and its validity dates are empty. (`TOF-049`)

---

## 9. Accrual plans

**AC-109.** Given an Accrual Plan Level whose milestone is reached "After" with an offset of zero,
when it is saved, then it is rejected with "You can not start an accrual in the past." (`TOF-080`)

**AC-110.** Given an Accrual Plan Level whose rate is zero, when it is saved, then it is rejected
with "You must give a rate greater than 0 in accrual plan levels." (`TOF-081`)

**AC-111.** Given a level whose action with unused accruals is "Carried over", whose carry-over
option is "Up to" and whose maximum is zero, when it is saved, then it is rejected with "You cannot
have a maximum quantity to carryover set to 0." (`TOF-082`)

**AC-112.** Given a level with the carried-over validity on and a count of zero, when it is saved,
then it is rejected with "You cannot have an accrual validity time set to 0." (`TOF-083`)

**AC-113.** Given a level with the yearly cap on and an amount of zero, when it is saved, then it is
rejected with "You cannot have a cap on yearly accrued time without setting a maximum amount."
(`TOF-084`)

**AC-114.** Given a level with the running cap on and a maximum of zero, when it is saved, then it is
rejected with "You cannot have a balance cap on accrued time set to 0." (`TOF-085`)

**AC-115.** Given a level whose frequency is Weekly with no weekday chosen, when it is saved, then it
is rejected with "Weekday must be selected to use the frequency weekly" (`TOF-086`)

**AC-116.** Given a level whose frequency is Twice a month with a first day of twenty and a second
day of five, when it is saved, then it is rejected with "The first day must be lower than the second
day." (`TOF-087`)

**AC-117.** Given a plan that grants at the start of the accrual period, when a level whose frequency
is Per Hour Worked is saved on it, then it is rejected with "You can't base accrued time on hours
worked, because time is accrued at the start of the period." (`TOF-088`)

**AC-118.** Given a plan carrying a level whose frequency is Per Hour Worked, when the plan is
switched to granting at the start of the accrual period, then the level's frequency becomes Hourly.

**AC-119.** Given an Accrual Plan referenced by an accrual allocation in state *Approved*, when it is
deleted, then it is rejected with "Some of the accrual plans you're trying to delete are linked to an
existing allocation. Delete or cancel them first." (`TOF-089`)

**AC-120.** Given a monthly level anchored on day one, when the next boundary is computed from the
sixteenth of January 2024, then it is the first of February 2024; when the previous boundary is
computed from the same date, then it is the first of January 2024; and given a monthly level anchored
on day fifteen, when the previous boundary is computed from the tenth of January 2024, then it is the
sixteenth of December 2023.

**AC-121.** Given a weekly level anchored on Monday, when the next boundary is computed from Monday
the first of January 2024, then it is Monday the eighth of January 2024; and when the previous
boundary is computed from Thursday the fourth of January 2024, then it is Monday the first of January
2024.

**AC-122.** Given a plan whose carry-over anchor is the start of the year, when the cut-off is
computed for the fifth of March 2024, then it is the first of January 2025; and when it is computed
for the first of January 2024, then it is the first of January 2024.

**AC-123.** Given a plan with a single monthly milestone granting 1.25 days anchored on day one, a
running cap of fifteen days, the action "Carried over" limited to five days, granting at the end of
the period, with the carry-over anchor at the start of the year, and an approved accrual allocation
of Bob valid from the first of November 2023 with no end date starting at zero, when the daily
accrual run has executed every day up to the thirty-first of December 2024, then Bob's allocation
carries exactly 15.00 days, having been granted 1.25 on the first of December 2023 and on the first
day of every month from January to November 2024, and having been refused the grant of the first of
December 2024 by the running cap.

**AC-124.** Given the state of AC-123, when the accrual run executes on the first of January 2025,
then the carry-over reduces the balance to 5.00 days before the December period grants 1.25, ending
at 6.25 days.

**AC-125.** Given the plan of AC-123 and an approved absence of four days taken in July 2024 charged
against the allocation, when the run has executed up to the thirty-first of December 2024, then the
gross amount is 16.25 days, the consumed amount is four days and the available balance is 12.25 days,
the running cap applying to the available balance and not to the gross amount.

**AC-126.** Given a plan granting at the start of the accrual period with a daily milestone granting
one day starting one day after the allocation start, a running cap of twenty-five days and the action
"Carried over" limited to fifteen days at the start of the year, and an accrual allocation valid from
the fifteenth of December 2021 starting at ten days and approved the same day, when the accrual run
executes with a target date of the first of January 2022, then the allocation carries exactly sixteen
days.

**AC-127.** Given a plan with two monthly milestones, the first granting one day from allocation
creation and the second granting two days after thirteen months, with the transition mode
"Immediately", and an allocation valid from the first of January 2023, when the run reaches the first
of February 2024, then the grant of that date is one day, and from the first of March 2024 onwards
each grant is two days.

**AC-128.** Given the same plan whose second milestone is Yearly on the first of January and whose
transition mode is "After this accrual's period", when the run reaches the first of March 2024, then
the first milestone still grants one day on that date, the second milestone's next boundary from the
transition date being the first of January 2025, later than the first milestone's next boundary of
the first of March 2024.

**AC-129.** Given a plan whose milestone carries a carried-over validity of three months, with the
carry-over anchor at the allocation date and the action "Carried over" with no limit, and an
allocation valid from the fifteenth of January 2023 that has accrued one day a month, when the run
reaches the fifteenth of April 2024, then the carried-over pool recorded on the fifteenth of January
2024 expires and the balance drops to the three days accrued since the cut-off.

**AC-130.** Given the same fixture with four days consumed before the expiry date, when the run
reaches the fifteenth of April 2024, then the expiring amount is the pool minus the consumed amount,
seven, and the balance drops from fourteen to seven.

**AC-131.** Given an accrual allocation in state *To Approve* on the form, when the validity start
date is changed, then the cursor is reset and the plan is replayed to today, showing the amount the
employee would already have accrued, before anything is saved. (`TOF-068`)

**AC-132.** Given an accrual allocation whose plan has not started because the first milestone starts
after today, when the accrual run executes, then nothing is granted and the next call date stays
empty. (`TOF-159`)

**AC-133.** Given an accrual allocation on its first run, when the engine initialises it, then an
internal note is posted reading "This allocation have already ran once, any modification won't be
effective to the days allocated to the employee. If you need to change the configuration of the
allocation, delete and create a new one."

**AC-134.** Given a plan with a yearly cap of ten days and a monthly milestone granting two days,
when the run has executed for seven months inside one carry-over period, then the total granted in
that period is ten days and the grants of the sixth and the seventh month are zero.

**AC-135.** Given a plan based on worked time with a monthly milestone granting four days, and an
employee who was absent for five working days of a twenty-two working day month on a type counted as
Absence and not eligible for the accrual rate, when the month closes, then the factor is
136 ÷ 176 = 0.772727 and the grant is 3.090909 days, printed as 3.09.

**AC-136.** Given a plan with a single monthly milestone granting 1.5 days anchored on day one, a
running cap of twenty days, the action "Carried over" limited to five days, granting at the end of the
period, with the carry-over anchor at the start of the year, and an approved accrual allocation valid
from the first of January 2024 starting at zero, when the engine runs with a target date of the first
of March 2025, then the balance is 9.5 days, twenty-one days having been accrued in fourteen grants
and eleven and a half having been lost at the cut-off of the first of January 2025.

**AC-137.** Given the allocation of AC-136 on the fifteenth of June 2024, when the future accrual is
projected to the first of October 2024, then the projection is 6.0 days and the maximum allowed
published for that date is 13.5 days.

---

## 10. Public holidays, mandatory days and optional holidays

**AC-138.** Given a public holiday on the first of May 2024 for the company and an approved request
of Alice from the twenty-ninth of April to the third of May of a type that excludes public holidays,
when the balance is read, then the request costs four days, not five.

**AC-139.** Given the same request, when the public holiday is deleted, then the request is restated
to five days, Alice is notified with "Due to a change in global time offs, 1.0 extra day(s) have been
taken from your allocation. Please review this leave if you need it to be changed.", and the request
returns to its previous state.

**AC-140.** Given a request of five days and no entitlement left, when a public holiday is created
inside its period, then the request is restated to four days and Alice is notified with "Due to a
change in global time offs, you have been granted 1.0 day(s) back."

**AC-141.** Given a request whose restatement makes it exceed the available entitlement, when the
public-holiday change is applied, then the request is refused and Alice is notified with "Due to a
change in global time offs, this leave no longer has the required amount of available allocation and
has been set to refused. Please review this leave." (`TOF-127`)

**AC-142.** Given a request of the shipped Sick Time Off type whose duration grows because a public
holiday was removed, when the change is applied, then no "extra day(s) have been taken" notification
is sent. (`TOF-126`)

**AC-143.** Given a public holiday for the company covering the first of May 2024, when a second
public holiday for the same company covering the same day is created with no working schedule, then
it is rejected with "Two public holidays cannot overlap each other for the same working hours."
(`TOF-120`)

**AC-144.** Given a public holiday covering the first of May 2024 attached to the schedule "Standard
40 hours", when a second one covering the same day is attached to a different schedule, then it is
accepted.

**AC-145.** Given an actor two hours ahead of coordinated universal time creating a public holiday
for a schedule whose time zone is four hours behind, spanning the whole of the fourth of July 2024 as
the actor types it, when the record is created, then the stored instants describe the whole of the
fourth of July 2024 in the **schedule's** time zone. (`TOF-123`)

**AC-146.** Given a Working Time Exclusion created with an empty end instant on the fourth of July
2024, when it is saved, then its end instant becomes the fourth of July at twenty-three hours,
fifty-nine minutes and fifty-nine seconds in the reader's time zone. (`TOF-122`)

**AC-147.** Given a Mandatory Day from the second to the sixth of September 2024 with no department
and no job position restriction, when Alice creates a request covering the fourth of September, then
it is rejected with "You are not allowed to request time off on a Mandatory Day" (`TOF-071`)

**AC-148.** Given the same Mandatory Day, when Olivia, an Officer, creates the same request for
Alice, then it is accepted. (`TOF-072`)

**AC-149.** Given a Mandatory Day restricted to the department "Sales", when Bob, who belongs to
"Support", creates a request covering it, then it is accepted; and when Alice, who belongs to a child
department of "Sales", creates the same request, then it is rejected. (`TOF-070`)

**AC-150.** Given a Mandatory Day restricted to the job position "Developer", when an employee with
no job position at all creates a request covering it, then it is rejected, an employee without a job
position ignoring the job restriction.

**AC-151.** Given a Mandatory Day whose start date falls after its end date, when it is saved, then
it is rejected with "The start date must be anterior than the end date." (`TOF-074`)

**AC-152.** Given a window and an employee, when the special-days operation is called, then it
returns the Mandatory Days with their colour index and the public holidays with colour index zero,
each as a whole-day block carrying a negative identifier.

**AC-153.** Given the Indian localization package installed and a type limited to optional holidays,
when a request of that type covers a day that is not declared as an Optional Holiday, then it is
rejected with "The following leaves are not on Optional Holidays:" followed by one line per offending
request. (`TOF-075`)

**AC-154.** Given an Optional Holiday that a request of such a type covers, when it is deleted, then
the deletion is rejected with "You cannot delete an Optional Holiday that is linked to a leave
request." (`TOF-076`)

**AC-155.** Given an acting company whose country is not India, when the creation form of an Optional
Holiday is opened, then it is refused with "You must be logged in an Indian company to use this
feature" (`TOF-077`)

---

## 11. Employee versions, working schedules and departure

**AC-156.** Given Jules holds an approved request of twenty-two days from the first to the thirtieth
of June 2022, and the running contract is closed on the fifteenth of June and a new Employee Version
starting the sixteenth of June with a different working schedule is created, when the version is
saved, then three requests exist for Jules: the original, now *Refused*; one from the first to the
fifteenth of June in state *Approved* carrying eleven days; and one from the sixteenth to the
thirtieth of June in state *To Approve* carrying eleven days. Each new request's thread carries an
origin link back to the original.

**AC-157.** Given Jules holds an approved request of four days from the twenty-seventh to the
thirtieth of June 2022, entirely after the start of a new Employee Version created on the sixteenth
of June with a different working schedule, when the version is saved, then exactly one request exists
for Jules, it is in state *To Approve*, and it still carries four days, a single version overlapping
it and no split being needed.

**AC-158.** Given Jules holds an approved request entirely inside the period of a single existing
version, when that version's working schedule is written directly, then the request is not split: it
adopts the new schedule, its instants are recomputed and, the write path checking the overlapping
versions before touching the state, it is revalidated and its Working Time Exclusion is rebuilt.

**AC-159.** Given a version change that would make an existing absence exceed the available
entitlement, when the version is saved, then it is rejected with a message beginning "Changing the
contract on this employee changes their working schedule in a period they already took leaves."
(`TOF-029`)

**AC-160.** Given an existing absence and Alice's working schedule being changed directly on the
employee record in a way that makes a future absence exceed the entitlement, when the employee is
saved, then it is rejected with "Changing this working schedule results in the affected employee(s)
not having enough leaves allocated to accomodate for their leaves already taken in the future. Please
review this employee's leaves and adjust their allocation accordingly." (`TOF-029`)

**AC-161.** Given an absence spanning two Employee Versions carrying different working schedules,
when its absolute instants are written, then it is rejected with the multi-version message of
`TOF-025`, which lists each overlapping version with its start and end date and prints the literal
word "undefined" for an empty end date.

**AC-162.** Given Alice's department changes from Support to Sales, when the employee is saved, then
every request of Alice that is *To Approve* or starts in the future receives the department Sales,
and every allocation of Alice that is *To Approve* receives the department Sales and the new manager.
(`TOF-154`)

**AC-163.** Given Alice holds an approved request from the fifth to the sixteenth of August 2024 and
her departure date is set to the ninth of August, when the departure is registered, then the original
request ends on the ninth of August and receives the comment "End date has been updated because the
employee will leave the company on `<departure date>`.", and the remainder from the twelfth to the
sixteenth of August is cancelled with the reason "The employee will leave the company on `<departure
date>`." without notifying the responsible approvers.

**AC-164.** Given Alice holds a request in state *To Approve* entirely after her departure date, when
the departure is registered, then that request is deleted rather than cancelled.

**AC-165.** Given Alice holds an allocation valid from the first of January 2024 with no end date and
her departure date is the ninth of August 2024, when the departure is registered, then the
allocation's validity end date becomes the ninth of August 2024 and it receives the comment "Validity
End date has been updated because the employee will leave the company on `<departure date>`."

**AC-166.** Given Alice holds an allocation whose validity starts after her departure date, when the
departure is registered, then that allocation is deleted, the allocation state guard being suppressed
for the departure procedure. (`TOF-067`)

**AC-167.** Given an hour-based allocation of forty hours held by an employee on an eight-hour day,
when a new Employee Version moves the employee to a six-hour day, then the stored day amount becomes
6.666667 and the stored hour amount stays forty. (`TOF-069`)

---

## 12. Multi-employee generation

**AC-168.** Given Olivia opens the Multiple Requests dialog with the mode By Company, the company
Main, the type Sick Time Off and the period from the twenty-fourth to the twenty-fourth of December
2024, when she confirms, then one approved request of one day is created for every employee of Main
who has working time that day, and employees with no working time that day are skipped.

**AC-169.** Given Mark, who is only a Time Off Responsible, opens the Multiple Requests dialog with
the mode By Company, when he confirms, then it is rejected with "As Time Off Responsible, you can only
use the allocation mode 'By Employee'." (`TOF-095`)

**AC-170.** Given Bob already holds a request in state *To Approve* from the twenty-third to the
twenty-seventh of December 2024 and Olivia generates a company-wide absence for the twenty-fourth of
December, when she confirms, then Bob's request is split into one covering the twenty-third and one
covering the twenty-fifth to the twenty-seventh, both keeping the state *To Approve*, and the new
one-day absence is created.

**AC-171.** Given Bob already holds a request in state *To Approve* covering exactly the twenty-fourth
of December 2024, when the same generation runs, then Bob's request is refused instead of being
split.

**AC-172.** Given Bob already holds an hour-based request overlapping the generated period, when the
generation runs, then the whole operation is rejected with a message beginning "Some employees already
have time off requests in hours that overlap with the selected period" and listing the conflicting
request. (`TOF-096`)

**AC-173.** Given Mark, a Time Off Responsible, generates a request for an employee he approves with a
type whose ladder is By Time Off Officer, when he confirms, then the created request is in state *To
Approve*, not *Approved*, Mark not being an Officer.

**AC-174.** Given a split whose second fragment would start after it ends, when the generation runs,
then that fragment is discarded rather than created. (`TOF-139`)

**AC-175.** Given Olivia opens the New Group Allocation dialog with the mode By Employee Tag, a tag
covering Alice and Bob, a regular allocation of ten days and the type Paid Time Off, when she
confirms, then two allocations of ten days are created and both are *Approved*, Olivia being an
Officer and the ladder being By Time Off Officer.

**AC-176.** Given the same dialog with an accrual allocation, a plan and an amount of zero, when she
confirms, then the created allocations are reset and replayed by the accrual engine from the validity
start date up to the earlier of the validity end date and today.

**AC-177.** Given Mark, a Time Off Responsible, uses the New Group Allocation dialog in By Employee
mode with a type whose ladder is By Employee's Approver, when he confirms, then the created
allocations are approved; and with a type whose ladder is By Time Off Officer, they stay in state *To
Approve*.

---

## 13. Access rights and multiple companies

**AC-178.** Given Bob, an internal user, when he creates a request naming Alice as the employee, then
the creation is rejected by the record rules. (`TOF-090`)

**AC-179.** Given Olivia, an Officer, when she creates a request naming Bob as the employee, then the
creation succeeds.

**AC-180.** Given a request of Bob, when Alice reads its description, then she reads the five
characters `*****`; when Bob reads his own, he reads the stored text; when Mark, Bob's Approver,
reads it, he reads the stored text; and when Olivia reads it, she reads the stored text. (`TOF-093`)

**AC-181.** Given a request of Bob, when Alice searches requests by description, then Bob's request
never matches, a non-Officer's description search being restricted to their own requests.

**AC-182.** Given a request of Alice in state *Approved*, when Alice writes any field on it, then the
write is refused by the record rules; and when Mark, her Approver, writes on it, then the write
succeeds under the responsible rule.

**AC-183.** Given a request of Alice in state *Approved*, when Alice posts a message on its thread,
then the message is posted, subscribing and posting on an approved record running with elevated
rights after a read-access check. (`TOF-098`)

**AC-184.** Given an Officer whose own request is in state *Approved*, when that Officer writes on it,
then the write is refused, the officer create-and-write rule excluding their own approved request.

**AC-185.** Given companies Main and Branch and a user allowed only Main, when that user lists
requests, then requests whose company is Branch are invisible. (`TOF-092`)

**AC-186.** Given a Time Off Type with no company and the country France, and a user whose allowed
companies are all Belgian, when that user lists types, then the French type is invisible.

**AC-187.** Given an Accrual Plan with no company, when any user lists plans, then it is visible.

**AC-188.** Given Mark is named as Alice's Time Off Approver, when the employee is saved, then Mark's
login user gains the Time Off Responsible group. (`TOF-094`)

**AC-189.** Given Mark approves only Alice and Alice's Approver is changed to Olivia, when the
employee is saved, then Mark loses the Time Off Responsible group.

**AC-190.** Given a company whose country is Belgium and an existing request of a Belgian-bound Time
Off Type, when the company's country is changed to France, then it is rejected with "The company
country cannot be changed while time off leaves or allocations with the country exist." (`TOF-099`)

**AC-191.** Given an employee is archived, when the archiving completes, then the Time Off Approver
link on that employee is emptied and the employee record can still not be deleted while any request
or allocation references it. (`TOF-155`)

---

## 14. The companion packages

**AC-192.** Given the timesheet companion package installed, the company carrying an internal project
and a time off task, and Alice's request from the thirteenth to the fifteenth of May 2024 of a type
counted as Absence being validated, when the validation completes, then three Analytic Lines exist,
one per working day, named "Time Off (1/3)", "Time Off (2/3)" and "Time Off (3/3)", each carrying
eight hours, the internal project, the time off task, Alice's login user, Alice's employee record and
the request as the origin.

**AC-193.** Given those lines exist, when the request is refused, then all three are unlinked and
deleted and any missing public-holiday lines for the period are regenerated.

**AC-194.** Given those lines exist, when a user tries to delete one directly, then it is rejected
with "You cannot delete timesheets that are linked to time off requests. Please cancel your time off
request from the Time Off application instead." (`TOF-101`)

**AC-195.** Given a timesheet line generated by a public holiday, when a user tries to delete it, then
it is rejected with "You cannot delete timesheets that are linked to global time off." (`TOF-100`)

**AC-196.** Given a timesheet line linked to an absence, when a user tries to edit its quantity, then
it is rejected with "You cannot modify timesheets that are linked to time off requests. Please use the
Time Off application to modify your time off requests instead." (`TOF-102`)

**AC-197.** Given the company time off task, when a user tries to record a manual timesheet line on
it, then it is rejected with "You cannot create timesheets for a task that is linked to a time off
type. Please use the Time Off application to request new time off instead." (`TOF-103`)

**AC-198.** Given a request whose type is counted as Worked Time, when it is validated, then no
timesheet line is generated.

**AC-199.** Given the payroll companion package installed and Alice's type mapped to the work entry
type "Paid Time Off", when a request of Alice is validated, then work entries carrying that work entry
type are created for the absence days, attendance work entries fully inside the absence are archived,
and attendance work entries extending beyond it lose their absence link.

**AC-200.** Given those work entries exist, when the request is refused, then they are archived and
attendance work entries are regenerated for those days.

**AC-201.** Given a work entry linked to an absence, when it is written into the cancelled state, then
the absence is refused. (`TOF-105`)

**AC-202.** Given a validated work entry linked to an approved absence, when the cancellation
permission of that absence is evaluated, then it is false and the Cancel button is hidden.
(`TOF-106`)

**AC-203.** Given the attendance companion package installed, a type "Compensatory Hours" that deducts
extra hours and requires no allocation, and Alice holding twelve hours of approved compensable
overtime, when Alice requests five hours of that type, then the request is accepted and her unspent
compensable extra hours become seven.

**AC-204.** Given the same fixture, when Alice requests thirteen hours, then it is rejected with "You
do not have enough extra hours to request this leave"; and when Olivia makes the same request on
Alice's behalf, then it is rejected with "The employee does not have enough extra hours to request
this leave." (`TOF-110`)

**AC-205.** Given the same fixture and an allocation of a type that deducts extra hours, when the
allocation would take the unspent balance below zero, then it is rejected with "The employee does not
have enough overtime hours to request this leave." (`TOF-111`)

**AC-206.** Given an allocation in state *Approved* and a user who is not an Officer, when that user
edits its amount, then it is rejected with "Only an Officer or Administrator is allowed to edit the
allocation duration in this status." (`TOF-112`)

**AC-207.** Given a public holiday created over a day on which Alice recorded an attendance, when the
holiday is saved, then Alice's attendance overtime is recomputed.

**AC-208.** Given the remote-working companion package installed and Alice absent today, when her
presence icon is computed, then it shows the on-leave variant rather than her work location, labelled
with the status and the return date.

**AC-209.** Given Alice is on an approved absence of a type counted as Absence covering the present
instant, when her login user's online status is read, then it is the on-leave variant of whatever it
would otherwise have been, and her formatted display name is suffixed with the return date.

**AC-210.** Given Alice's absence ends on Friday the seventeenth of May 2024 at 17:00 local and the
next working interval begins on Monday the twentieth of May at 08:00 local, when her return date is
computed, then it is the twentieth of May 2024. (`TOF-137`)

**AC-211.** Given the French localization package installed, a company whose country is France whose
schedule is Monday to Friday, an employee working Monday to Wednesday, and a full-day request of the
company's reference type from Monday to Wednesday of one week, when the request is saved, then its
duration is five days, its end instant is extended to the Friday and the flag "end date extended by
the French rule" is true. See
[calculations.md, section 15.1](calculations.md#151-the-french-part-time-rule).

**AC-212.** Given that request validated and the French work entry package installed, when the work
entries are generated, then entries carrying the type's work entry type additionally cover the
Thursday and the Friday, one per company attendance interval on a date that carries no work entry
yet. (`TOF-109`)

**AC-213.** Given the French rule applying to an employee whose working schedule carries no attendance
line at all, when the request is saved, then it is refused with "An employee can't take paid time off
in a period without any work hours."

**AC-214.** Given the Indian localization package installed, an employee working Monday to Friday
holding a bridging-enabled absence on Friday the fifth of July, when a second bridging-enabled absence
is filed on Monday the eighth of July, then the second absence carries three days and twenty-four
hours and its flag "contains bridging days" is true.

**AC-215.** Given the same fixture, when the Friday absence is refused, then the Monday absence is
restated to one day and its bridging flag is cleared.

---

## 15. The dashboard, the reports and the scheduled jobs

**AC-216.** Given Alice holds the allocations of AC-075, when the dashboard is read for the first of
May 2024, then one tile exists for Paid Time Off showing seventeen available out of twenty-five, with
three requested and five taken, and the tile is not marked as holding changes when the evaluation date
is today.

**AC-217.** Given an accrual allocation that will have granted two more days by the first of September
2024, when the dashboard is read for the first of September 2024, then the accrual bonus is two, the
allocated amount includes it, and the tile is marked as holding changes.

**AC-218.** Given a Time Off Type flagged as hidden from the dashboard, when the dashboard is read,
then no tile exists for it, and the type is still offered in the request form.

**AC-219.** Given the Time Off Analysis projection, when it is read for an employee holding one
allocation of ten days and one request of three days, then two rows exist, the allocation row carrying
plus ten days and the request row minus three days, and the sum of the group is seven.

**AC-220.** Given the Time Off Balance by Employee and Type projection, when it is read for an employee
holding one approved allocation of ten days and one approved absence of three days, then one row of
kind Left carries seven days and one row of kind Taken carries three days.

**AC-221.** Given two approved allocations of the same employee and type with disjoint validity windows
and an absence overlapping only the second, when the balance projection is read, then the absence is
charged against the second allocation only and the first allocation's remaining balance is untouched.

**AC-222.** Given a department manager who manages the department Sales, when they read the Time Off
Analysis projection, then they see only the rows of the employees of Sales.

**AC-223.** Given the printed sixty-day summary launched for Alice with a start date of the first of
May 2024 and the state filter Approved, when the document is produced, then the header names the first
of May 2024 and the twenty-ninth of June 2024 and the word "Approved", every Saturday and Sunday cell
is shaded, every day of every approved absence of Alice intersecting the window is painted with the
colour of its type, and the row total is the sum of the day figures of those absences.

**AC-224.** Given the daily accrual run, when it executes, then it selects only accrual allocations
that are *Approved*, carry a plan and an employee, have not expired, and whose next call date has
arrived, and it advances each of them to today.

**AC-225.** Given Alice holds an approved request of five days starting in twenty days funded only by
an accrual allocation that will hold three days by then, when the daily invalid-absence run executes,
then the request is cancelled with the internal note "The time off request has been cancelled for the
following reason:" followed by "the accruated amount is insufficient for that duration." and the
responsible approvers are not notified. (`TOF-140`)

**AC-226.** Given Alice holds two such requests, one starting in ten days and one in twenty-five days,
and the accrual can fund only one, when the run executes, then the later one is cancelled first and
the earlier one survives. (`TOF-141`)

**AC-227.** Given an absence funded by an accrual allocation whose maximum allowed at the absence start
date is zero, when the run executes, then the absence is cancelled whatever the allowed excess.

**AC-228.** Given an approver follows the action link of a notification for a request they may not
approve, when they follow it, then they are redirected to a generic fallback page and the request is
unchanged.

**AC-229.** Given an approver follows the refuse action link with a valid token, when they follow it,
then the request is refused and they land on the request.

**AC-230.** Given an absence of five days starting after the evaluation date and funded by an accrual
allocation that will hold three days by then, when the balance is read at the evaluation date, then the
absence is not charged, no excess entry is recorded for it, and the exceeding duration is minus two.
(`TOF-058`)

---

## 16. Activities and notifications

**AC-231.** Given a request of Alice created on the first of May 2024 with an absolute start on the
third of June 2024 under the By Time Off Officer ladder, when it is created, then one Time Off Approval
activity is created for each notified officer with a deadline of the nineteenth of May 2024, the start
date minus the fifteen-day delay of that activity type.

**AC-232.** Given the same request with an absolute start on the fifth of May 2024, when it is created,
then the deadline is floored at the first of May 2024, the computed deadline falling in the past.

**AC-233.** Given a request under the By Employee's Approver ladder for an employee with no Time Off
Approver and no hierarchical parent, when it is created, then the activity is created for the notified
officers of the type.

**AC-234.** Given a request under the two-approval ladder in state *To Approve*, when the responsible
approvers are determined, then they are the employee's Time Off Approver; and when the request is in
state *Second Approval*, then they are the notified officers of the type.

**AC-235.** Given a request carrying activities, when it is cancelled, then both activity types are
removed, including any a user created by hand.

**AC-236.** Given an allocation created under a type whose **request** ladder is None needed, when it is
created, then no Allocation Approval activity is created, even when the allocation ladder itself
requires an approval. This is the compatibility finding recorded in
[workflows.md, section 5.3](workflows.md#53-activities-on-a-time-off-allocation).

**AC-237.** Given a notification subtype is created whose target is the Time Off Request, when it is
saved, then a mirrored subtype carrying the same name is created on the Department entity, pointing
back at it and relating through the department link.

**AC-238.** Given a request reaching *Approved* on a Time Off Type whose request notification subtype is
"Sick Time Off", when the state change is tracked, then the message is posted under that subtype
instead of the default one.

---

## 17. Reconciliation notes

1. **Only one draft carried this file.** Every one of its scenarios is retained, renumbered into one
   continuous sequence and re-pointed at the rule identifiers of
   [business-rules.md](business-rules.md#16-index-of-rule-identifiers).
2. **Scenarios added during the consolidation.** The other draft carried facts that no scenario
   covered: the deferred set and the exceeding duration (AC-230), the fourteen-month accrual example and
   its projection (AC-136 and AC-137), the country-specific duration rules (AC-211 to AC-215), the
   defaulting of a working time exclusion's end instant (AC-146), the officer's own approved request
   (AC-184), the negative-balance tile (AC-090), the discarding of a zero-length fragment (AC-174), the
   archiving of a type (AC-012), the freezing of the resolved period (AC-071) and the restatement of an
   hour-based allocation on a schedule change (AC-167). Each is written here with concrete numbers.
3. **The fixture.** One draft named a fixed company, four employees and four types; that fixture is kept
   and extended with the two localization scenarios, which name their own company country explicitly.
