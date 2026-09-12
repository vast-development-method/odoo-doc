# Work Entries — Accounting Effects

## 1. The direct answer

**This domain posts nothing to the ledger.** Not one operation of the Work Entries domain creates a
Journal Entry, a Journal Item, a tax line or an analytic line. Generation, validation, conflict
marking, regeneration, cancellation and deletion are all movements of hours, never of money.

The reasoning is worth stating explicitly, because a reader who has seen the pay rate on a work entry
may expect otherwise:

1. A work entry carries **hours**, a **kind** and a **pay rate multiplier**. It carries no amount, no
   currency and no account. There is no field on any entity of this domain that could hold a monetary
   value.
2. The pay rate is a multiplier, not a price. It says "these hours are worth one and a half times the
   ordinary rate", and the ordinary rate lives on the employment terms, which belong to
   [Human Resources Core](../human-resources-core/README.md), not here.
3. The domain has no notion of a period being *closed*. Its `validated` state means "a payroll run has
   taken this row", which is a lock, not a posting.

---

## 2. What the domain supplies to the capability that does post

A payroll capability consumes the day book and turns it into a payslip, and the payslip is what posts.
The contract between the two is small and is listed here item by item so that a rebuild knows exactly
what its ledger-producing capability may rely on.

| Supplied | Where it lives | What a payroll capability does with it |
|---|---|---|
| The payroll code of each row's kind | mirrored, read-only, onto the work entry from its kind | Selects the salary rule that prices those hours |
| The number of hours | the duration on the work entry | The quantity the salary rule multiplies |
| The calendar date | the date on the work entry | Assigns the hours to a payroll period |
| The employee and the employment version | two required links on the work entry | Selects the wage, the structure and the company |
| The pay rate | copied from the kind onto the work entry at creation and never recomputed | The multiplier the salary rule applies |
| The extra-hours flag | on the kind | Decides whether the amount is added on top of the basic salary or is part of it |
| The absence flag | on the kind | Decides whether the row reduces worked time or records paid or unpaid absence |
| The external code | on the kind | Lets an export use a third party's vocabulary rather than the internal payroll code |
| The company | required on the work entry, taken from the employee | Selects the journal and the chart of accounts on the payroll side |
| The absence kind behind an absence row | through the absence link, when the absence companion is installed | Lets an absence provision be recognised against the right kind |
| The per-kind absence total | the named operation of [interfaces.md, chapter 8](interfaces.md#8-named-operations) | Feeds a payslip's absence lines without re-reading every row |

The **stability** of the pay rate is the one property of this list that has a direct financial
consequence, and it is deliberate. The rate is copied onto the entry when the entry is created and is
never recomputed: changing a kind's rate does not move rows a payslip has already priced, and changing
a row's kind does not move its rate. A rebuild that recomputes the rate will silently restate closed
payroll periods.

---

## 3. The ledger effects this domain triggers indirectly

Every one of these is produced by another domain. This chapter names them so that a reader following
a figure from a payslip back to its origin knows where to look.

| Event here | Consequence elsewhere | Where it is specified |
|---|---|---|
| A period's rows reach the `validated` state | The payroll capability may price them; the payslip it produces posts a Journal Entry debiting salary expense and crediting the employee payable and the social contribution payables | The payroll capability of the rebuild; the entry itself is an ordinary Journal Entry as described in [General Ledger](../general-ledger/README.md) |
| The hours of an attendance row | Become part of the gross amount of a payslip line, which reaches the ledger as part of that payslip's entry | [General Ledger](../general-ledger/README.md) for the entry, [Analytic accounting](../analytic-accounting/README.md) for the distribution when the payroll capability distributes labour cost |
| The hours of an absence row of a paid kind | Become a paid-absence line of a payslip, priced at the kind's rate, posted with the same entry | As above |
| The hours of an absence row of an unpaid kind | Reduce the gross of the payslip, so the ledger effect is a smaller debit to salary expense | As above |
| The hours of an out-of-contract row | Reduce the gross for the days the employee had no contract | As above |
| The hours of an extra-hours kind | Become a bonus line added on top of the basic salary | As above |
| The absence hours grouped per absence kind | Feed the provisioning of accrued absence, where a payroll capability provides it | The payroll capability; the balance itself belongs to [Time Off](../time-off/README.md) |
| Cost per hour, where a project capability re-prices labour | The domain supplies none of this. Project and timesheet labour costing reads timesheet lines, not work entries | [Timesheets](../timesheets/README.md) |

---

## 4. What deliberately does not happen

Four negatives, each of which a reader might expect and none of which is true:

1. **Validating a work entry posts nothing.** It is a lock, taken so that regeneration, absence
   swallowing and deletion leave the row alone. No journal, no account, no amount.
2. **Cancelling or archiving a validated work entry reverses nothing.** Because the state and the
   archived flag are coupled, archiving a validated row cancels it; if a payslip has already priced
   those hours, nothing in this domain restates the payslip. That restatement, if it is wanted, is the
   payroll capability's business.
3. **A conflict has no financial meaning.** It blocks validation and therefore blocks a payroll run
   from taking the day, which is its whole purpose. It produces no provision and no entry.
4. **Multi-currency does not arise.** No quantity in this domain is monetary, so no rate is applied, no
   rate date is chosen and no exchange difference can occur. The company on a work entry selects the
   population and the record rule, not a currency. Where a payroll capability needs a currency it takes
   it from the company's own currency, which belongs to
   [Multi-currency](../multi-currency/README.md).

---

## 5. The audit trail this domain does provide

Although nothing is posted, three trails exist and a financial auditor will ask for them:

| Trail | What it records | Where |
|---|---|---|
| The change tracking on the version's markers | Every change to Generated From, Generated To and Last Generation Date is posted into the version's message thread with its old and new value | [Messaging and Activities](../messaging-and-activities/README.md) |
| The cancellation trail | A superseded row is never deleted by a forced regeneration; it is archived and cancelled and stays readable | [state-machines.md](state-machines.md#2-the-archived-flag-as-a-machine) |
| The deletion protection | A validated row cannot be deleted at all, and a human resources officer cannot delete any row | rules [`WKE-035`](business-rules.md#8-deletion-and-archiving-rules) and [`WKE-054`](business-rules.md#11-permission-checks) |

Between them they mean that the hours a payroll run priced can always be reconstructed: the validated
rows are still there, the rows they replaced are still there in the cancelled state, and the marker
history shows when each period was generated.
