# Time Off — Accounting Effects

This domain creates no journal entry, changes no journal item, moves no valuation and computes no
tax. Nothing in it touches the general ledger directly, and a rebuild may implement the whole domain
with no accounting capability present at all.

This file states that boundary precisely, itemises what the domain does write, and then describes
the indirect paths by which an absence eventually reaches the books. Each of those paths is owned by
another domain, and each is named with a link.

---

## 1. Why the domain posts nothing

Three properties together make a ledger effect impossible here.

1. **No field of this domain is monetary.** Every stored amount is a number of days or a number of
   hours. There is no currency link, no exchange rate, no rounding to a currency precision and no
   tax on any record of the domain, which is why
   [multi-currency](../multi-currency/README.md) and [taxes](../taxes/README.md) appear nowhere in
   the entity tables of [entities.md](entities.md).
2. **The cost of an absence is not known here.** What an absent day costs depends on the employee's
   remuneration, on the work entry type the absence maps to and on the salary rules that read it.
   None of those live in this domain.
3. **Entitlement is not a liability that this domain measures.** A balance is a quantity of days,
   not a valued obligation. Valuing it requires a cost per day, which again lives elsewhere.

---

## 2. Everything this domain writes

The complete list of records an approval, a refusal, a cancellation, a reopening or a deletion
writes:

| Record | Owning domain | Financial nature |
|---|---|---|
| Time Off Request, Time Off Allocation, Accrual Plan, Accrual Plan Level, Mandatory Day, Optional Holiday | Time Off | none |
| Working Time Exclusion (`resource.calendar.leaves`) | [Attendances and working time](../attendances-and-working-time/README.md) | none; it describes availability |
| Calendar Event | [Calendar and scheduling](../calendar-and-scheduling/) | none |
| Work Entry | [Work entries](../work-entries/README.md) | none by itself; it is an input of payroll |
| Analytic Line, the timesheet line | [Analytic accounting](../analytic-accounting/README.md) | analytic only, never a journal item |
| Scheduled activity, discussion message, follower | [Messaging and activities](../messaging-and-activities/README.md) | none |
| Attendance overtime recomputation | Attendances and working time | none |

None of these carries a debit, a credit, an account, a journal or a currency.

---

## 3. The analytic path

With the timesheet companion package installed, validating a Time Off Request creates one Analytic
Line per working day of the absence, on the acting company's internal project and on its time off
task, carrying the day's working hours as the quantity, the employee's login user, the employee and
the request as the origin. The analytic account of the line is the analytic account of the internal
project. The full procedure is
[workflows.md, section 3.1](workflows.md#31-materialisation-of-an-approved-absence), step 1.

Those lines are analytic only. They carry a quantity of hours and no monetary amount, they are not
journal items, and they are posted nowhere. Their financial consequences, where any exist, belong to
two other domains:

- the analytic distribution and the analytic cost of an employee hour, which
  [analytic accounting](../analytic-accounting/README.md) defines;
- the profitability and the cost roll-up of the internal project, which
  [timesheets](../timesheets/README.md) and [projects and tasks](../projects-and-tasks/README.md)
  define.

The lines are created and deleted only by this domain, and rules `TOF-100` to `TOF-104` of
[business-rules.md](business-rules.md#10-timesheet-lines-and-payroll-work-entries) forbid every
other actor from creating, editing or deleting them. That protection exists precisely because those
lines mirror an approval decision and must not drift from it.

---

## 4. The payroll path

With the payroll companion package installed, validating a Time Off Request creates Work Entries
carrying the work entry type configured on the Time Off Type, and archives the attendance work
entries the absence replaces. A Work Entry is not an accounting record: it is a quantity of time
with a payroll code attached.

The chain from there to the ledger is owned entirely by the payroll capability:

1. A payslip reads the work entries of its period and converts them into salary rule inputs
   according to the work entry type.
2. The salary rules compute the gross amounts, the employer and employee contributions and the net
   pay.
3. Posting the payslip, or posting the payroll journal entry built from a batch of payslips, is what
   creates the journal entry.

Consequently, a Time Off Type marked unpaid, or mapped to a work entry type whose salary rules pay
nothing, reduces pay through that chain and never through this domain. A rebuild that implements
Time Off without payroll simply stops after the work entry.

---

## 5. What this domain deliberately does not do

- **It books no liability for unused entitlement.** A provision for compensated absences that vest
  or accumulate is an accounting estimate built from the balances this domain publishes, and it
  belongs to the [general ledger](../general-ledger/README.md). This is an
  **industry-standard default**: the observed behaviour carries no such posting at all, and a
  rebuild that wants one should read the per-employee, per-type remaining balance of
  [calculations.md, chapter 7](calculations.md#7-aggregating-the-balance-for-a-date-and-the-dashboard-payload)
  at the reporting date, value it with the payroll capability's cost per day, and post the provision
  from the general ledger domain. The entry a rebuild would post is, per employee and per type: a
  debit to the personnel expense account of the employee's department for the valued balance, a
  credit to a current liability account for compensated absences for the same amount, in the
  company's currency at the reporting date, with the employee as the counterparty, the department's
  analytic distribution, no tax, and a reversal on the first day of the following period.
- **It recognises no expense when an absence is taken.** The expense follows the payslip.
- **It computes no cost in any currency.** No field is monetary, so no currency, no exchange rate and
  no currency rounding appears anywhere in the domain.
- **It reconciles nothing and has no reversal in the accounting sense.** Reversing an approval means
  refusing or cancelling the request, which removes the Working Time Exclusion, archives the Calendar
  Event, deletes the timesheet lines and archives the work entries, and lets payroll regenerate its
  own inputs.

---

## 6. Consequences for a rebuild

A rebuild of this domain needs no chart of accounts, no journal, no fiscal position, no tax and no
currency table. It needs:

1. the attendances and working time capability, to write Working Time Exclusions;
2. optionally the analytic accounting capability, when timesheet lines are wanted;
3. optionally the work entry machinery and a payroll capability, when absence must reach a payslip.

If none of the optional pieces exists the domain is still fully functional: requests are approved,
balances are tracked, calendars are populated, and nothing is ever posted.

---

## 7. Reconciliation notes

1. **Only one draft carried this file.** Its boundary statement is kept in full. The itemisation of
   [chapter 2](#2-everything-this-domain-writes) is extended with the Optional Holiday, which the
   other draft added to the entity list.
2. **The liability provision.** One draft stated the absence of a provision as a plain fact; it is
   kept here and marked as an **industry-standard default**, with the entry a rebuild would post
   itemised in [chapter 5](#5-what-this-domain-deliberately-does-not-do), as rule nine of the
   documentation rules requires of any ledger effect a specification describes.
