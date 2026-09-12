# Attendances and Working Time — Accounting effects

This domain posts **no journal entry**. It creates no journal item, holds no account, computes
no tax, carries no currency and touches no analytic distribution of its own. It records
**time**, not money. A rebuild can implement the whole domain with no general ledger present at
all, and the acceptance criteria of [acceptance-criteria.md](acceptance-criteria.md) never
mention an amount of money except in the one comparison analysis described below.

This file states why that is so, what the domain hands to the domains that do post entries, and
what a rebuild must preserve so that those domains keep working.

---

## 1. Why an attendance is not a valuation event

Three properties place presence outside the ledger.

1. **An attendance is not an obligation.** Checking in creates no liability towards the
   employee. The obligation arises from the employment contract and is recognised by payroll on
   its own period, whether or not the employee checked in. A day recorded twice and a day not
   recorded at all both leave the contractual obligation untouched.
2. **An attendance is mutable until it is consumed.** Records are corrected by officers, closed
   by an unattended job, created by another unattended job and regenerated wholesale whenever a
   rule, an exclusion or a neighbouring record changes. A ledger entry must be immutable once
   posted, so the recognition point is placed downstream.
3. **Extra hours are approvable, not merely measurable.** The quantity that eventually reaches
   payroll is the **encoded** amount of an **approved** line, which an approver may set below or
   above the computed amount. Until that human decision exists, no amount is determinable, and a
   regeneration can return an already-approved day to the pending state.

---

## 2. What the domain produces that money is later derived from

| Produced here | Where it is specified | Consumed by | What the consumer does with it |
|---|---|---|---|
| Worked hours per attendance | [calculations.md, chapter 3](calculations.md#3-the-worked-hours-of-one-attendance) | payroll, through [Work Entries](../work-entries/) | Reconciles declared presence against the work entries of the period. |
| Regular hours per attendance | [calculations.md, chapter 4](calculations.md#4-regular-hours-extra-hours-and-validated-extra-hours) | payroll and the attendance analysis | Separates the part of a day paid at the ordinary rate from the part that is not. |
| Validated extra hours per attendance, and the balance per employee | [calculations.md, chapter 5.3](calculations.md#53-total-approved-extra-hours-of-an-employee) | payroll, and the deductible balance of [Time Off](../time-off/) | Pays the quantity at the combined rate carried on the line, or converts it into absence entitlement. |
| The combined pay rate on a line | [calculations.md, chapter 7](calculations.md#7-combining-pay-rates) | payroll | Multiplies the ordinary hourly rate for the hours of that line. A rate of `1.5` means one hundred and fifty per cent of the ordinary rate; a rate of `0` means the line is not paid at all and exists only as a record or as an entitlement. |
| The compensable-as-time-off flag on a line | [entities.md, chapter 9.2](entities.md#92-field-table) | [Time Off](../time-off/) | Feeds the deductible balance that lets an employee take absence against banked extra hours. |
| Working intervals, working days and working hours between two instants | [working-schedule-algorithms.md](working-schedule-algorithms.md) | payroll, [Time Off](../time-off/), [Work Entries](../work-entries/), [Projects and Tasks](../projects-and-tasks/), [Manufacturing](../manufacturing/) | Converts a duration expressed in days into one expressed in hours and back, which is the basis of every entitlement, of every day-based pay element and of every work order's expected duration. |
| Working Time Exclusions | [entities.md, chapter 5](entities.md#5-working-time-exclusion) | [Work Entries](../work-entries/) | Slices the intervals into typed entries, which are what payroll values. |

None of those hand-overs happens inside this domain: it writes only the entities it owns. Every
consumer reads the records through the platform.

---

## 3. The ledger effects this domain triggers indirectly

The chain from a check-in to a journal entry has three links, and each belongs to another
folder.

1. **Presence becomes work entries.** [Work Entries](../work-entries/) walks the working
   intervals this domain produces and splits them by exclusion type into typed entries. That
   domain owns the typing; this one owns the intervals.
2. **Work entries become payslip lines.** Payroll values the typed entries and the approved
   extra-hours quantities at the rates its own rules carry. The combined rate of a line is a
   multiplier of an hourly rate, not an amount: this domain never states a currency.
3. **Payslip lines become journal items.** The ledger entry is posted by the payroll domain, in
   the journal, on the accounts and at the date that domain decides. Nothing in this folder
   constrains any of those choices.

The one indirect effect that does **not** pass through payroll is analytic: timesheet lines
carry an analytic distribution and reach the ledger through
[Projects and Tasks](../projects-and-tasks/) and
[Analytic Accounting](../analytic-accounting/). Attendance records carry no analytic
distribution and never create a timesheet line; the comparison analysis of the next chapter
only places the two side by side.

---

## 4. The one place where a monetary figure appears

The comparison analysis multiplies hours by the employee's hourly cost, so that a cost figure
can be shown beside a time figure:

```formula
attendance_cost = attendance_time in hours × hourly cost of the employee
timesheet_cost  = timesheet_time  in hours × hourly cost of the employee
cost_difference = ( attendance_time − timesheet_time ) in hours × hourly cost of the employee
```

Each of the three figures is reported as **empty rather than zero** when the product is zero.

The analysis is read-only and derived: it stores nothing, posts nothing, and carries no currency
field of its own. The hourly cost belongs to
[Human Resources Core](../human-resources-core/) and is expressed in the currency of the
employee's company; a reader who compares two companies compares two currencies without a
conversion, and the analysis does not attempt one. The analytic consequences of the timesheet
side belong to [Timesheets](../timesheets/).

**Worked example.** An employee whose hourly cost is thirty records eight presence hours and
seven timesheet hours on one day. The row reports a timesheet cost of 7 × 30 = **210**, a
presence cost of 8 × 30 = **240** and a cost difference of ( 8 − 7 ) × 30 = **30**. Had the two
times been equal, the cost difference would be reported as empty rather than as zero.

---

## 5. Reversal behaviour

There is nothing to reverse in an accounting sense, because nothing was posted. The equivalent
operations are these, and a rebuild must reproduce their effect on the downstream consumers.

| Event | Effect on the records of this domain | Effect downstream |
|---|---|---|
| An attendance is deleted | Every extra-hours line of the affected local days is deleted and rebuilt from the records that remain | The employee's balance changes at once; payroll must re-read it before the next run |
| An attendance is corrected | The union of the window before and the window after is rebuilt | The same |
| A rule or a rule set is changed and its regeneration action is run | Every line of every attendance governed by that rule set is deleted and rebuilt; days a person had touched come back pending | The same, and an amount that was approved may cease to be |
| A Working Time Exclusion is created, moved or deleted | Every affected day is rebuilt | The same, and the working intervals other domains read change immediately |
| Either company tolerance amount is changed | Every attendance of every employee of that company is rebuilt | The same, across the whole company at once |
| An employee is archived | The open record is closed at the current instant | One more closed record enters the next payroll period |

Because a regeneration is a delete-and-recreate, **the identifiers of the extra-hours lines are
not stable across regenerations**. A downstream consumer must join on the employee, the local
day and the attendance's two instants rather than on the line identifier. This is an explicit
constraint on any rebuild, and it is the reason a line stores the check-in and check-out of its
attendance rather than the boundaries of its own stretch.

---

## 6. Completions stated for a rebuild

The observed behaviour leaves the hand-over to payroll implicit. The following completions are
stated so that a rebuild can be built without ambiguity. Each is an **industry-standard
default**, not an observed decision of the system.

1. **Recognition point.** Extra hours are recognised as a payroll cost in the period in which
   the attendance's local day falls, not in the period in which the approval happens. An
   approval granted after that period has been paid produces a correction in the next period
   rather than a restatement of the paid one. **Industry-standard default.**
2. **Unapproved extra hours.** Computed but unapproved extra hours are not an obligation and are
   not accrued. Where local law or a collective agreement makes worked overtime payable
   regardless of approval, the appropriate configuration is to set the company's extra-hours
   validation to "Automatically Approved", which is exactly what that setting expresses.
   **Industry-standard default.**
3. **Banked extra hours.** Extra hours flagged compensable as time off represent a liability of
   the employer measured in **hours**, not in currency. Its monetary measurement, where a
   jurisdiction requires one, is the responsibility of payroll and uses the employee's rate at
   the reporting date. **Industry-standard default.**
4. **Negative lines.** A shortfall reduces the employee's balance but never creates a receivable
   from the employee. Recovering unworked time is a payroll matter or a working-time-account
   matter and is outside this domain. **Industry-standard default.**
5. **Unpaid rules.** A line whose combined rate is zero — which is what every rule of the
   shipped country rule set produces, none of them being flagged paid — is a record of time, not
   of money, and payroll values it at nothing unless the installation sets the paid flag itself.
   **Industry-standard default.**

---

## 7. Reconciliation notes

One source version carried this file; the other stated, inside its introduction, only that the
domain posts no journal entry. The two were merged and three points were added.

1. **The indirect chain was named in full.** Saying that the domain posts nothing is not enough
   for a rebuild: [chapter 3](#3-the-ledger-effects-this-domain-triggers-indirectly) names the
   three links from a check-in to a journal item and says which folder owns each.
2. **The currency of the comparison analysis.** Neither version said which currency the cost
   figures are in. They are in the currency of the employee's company, no conversion is
   performed, and a reader comparing two companies compares two currencies; that is stated in
   [chapter 4](#4-the-one-place-where-a-monetary-figure-appears).
3. **The instability of line identifiers** was stated by one version as a remark and is raised
   here to an explicit constraint on downstream consumers, with the join key named.
