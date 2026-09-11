# Accounting effects of the Human Resources Core domain

## 1. Statement

**This domain produces no journal entries.**

No operation described anywhere in this folder — creating an employee, creating or archiving
an Employee Version, opening or closing a contract, registering a departure, recording a
skill, assigning equipment, declaring a home-working day, or granting a badge — writes a
Journal Entry (`account.move`, table `account_move`) or a Journal Item (`account.move.line`,
table `account_move_line`). The domain stores no account, no journal, no tax and no analytic
distribution of its own, and it never calls the posting machinery.

The wage carried on an Employee Version is a reference figure, not a liability: recording a
wage of 3 500 creates no payable, no accrual and no expense. Likewise the salary
distribution across bank accounts is a payment instruction template, not a payment.

## 2. What the domain contributes to other domains that do post

Although it posts nothing, this domain is the source of several values that other domains
turn into journal entries. A reimplementation must get these values right, because an error
here becomes a financial error elsewhere.

### 2.1 The hourly cost, consumed by timesheets and project profitability

| Value supplied | Where it is defined | Who consumes it |
|---|---|---|
| Hourly Cost (`hourly_cost`), a monetary amount on the Employee, expressed in the company currency, defaulting to zero | [entities, section 2.14](entities.md#214-field-table--equipment-and-cost) | The [timesheets](../timesheets/README.md) domain multiplies it by the hours recorded to obtain the cost of a timesheet line; the [projects and tasks](../projects-and-tasks/README.md) domain aggregates those costs into project profitability. |

```formula
timesheet_line_cost = − ( hours_recorded × employee_hourly_cost )
```

The sign convention (a cost is negative) and the rounding to the company currency belong to
the consuming domain. What this domain guarantees is:

- the figure is per hour, not per day or per month;
- the figure is expressed in the **company's** currency, taken from the employee's company;
- the figure is a plain stored value, **not** derived from the wage — see
  [calculations, section 17.4](calculations.md#174-hourly-cost).

The consequence for a reimplementation is that the hourly cost must be maintained
independently of the wage. Changing the wage on a version does not change the hourly cost, and
nothing in this domain reconciles the two.

### 2.2 The wage and the contract dates, consumed by payroll

| Value supplied | Where it is defined | Who consumes it |
|---|---|---|
| Wage (`wage`) and Contract Wage (`contract_wage`) | [entities, section 3.8](entities.md#38-field-table--remuneration-and-classification) | A payroll capability, which turns them into salary rules and posts the resulting journal entries. |
| Salary Structure Type (`structure_type_id`) | same | A payroll capability, which selects the rule set from it. |
| Contract Start Date and Contract End Date | [entities, section 3.7](entities.md#37-field-table--the-contract-period) | A payroll capability, to determine the period being paid; the [work entries](../work-entries/README.md) domain, to bound generation. |
| The salary cost factor, the constant twelve | [calculations, section 17.3](calculations.md#173-salary-cost-factor) | A payroll capability, to annualise a monthly wage. |
| The normalised wage | [calculations, section 17.2](calculations.md#172-normalised-wage-an-hourly-equivalent) | Any consumer needing an hourly equivalent of a monthly wage. |

### 2.3 The salary distribution, consumed by payment

| Value supplied | Where it is defined | Who consumes it |
|---|---|---|
| Bank Accounts and the Salary Distribution map | [entities, section 2.8](entities.md#28-field-table--bank-accounts-and-salary-distribution) | A payroll or payment capability, which splits the net pay across the accounts according to the map and produces one outgoing payment per account. |
| The outgoing-payment permission of the primary bank account | same | The [payments and bank reconciliation](../payments-and-bank-reconciliation/README.md) domain, which refuses to pay to an account that is not permitted. |

The map's semantics matter financially and are specified in
[calculations, section 10](calculations.md#10-salary-distribution-across-bank-accounts): an
entry is either a **percentage** of the amount to pay or a **fixed amount** in the account's
currency (falling back to the company currency), and when any percentage entry exists the
percentages must sum to exactly 100.

### 2.4 The employment period, consumed by time off and work entries

| Value supplied | Who consumes it |
|---|---|
| Whether the employee is under contract on a date, and the contract period bounds | The [time off](../time-off/README.md) domain, to refuse or bound a leave; the [work entries](../work-entries/README.md) domain, to bound generation; the [attendances and working time](../attendances-and-working-time/README.md) domain, to decide expected hours. |
| The working schedule in force over each sub-period | All three of the above, plus the [calendar and scheduling](../calendar-and-scheduling/README.md) domain. |

Work entries and validated time off are the input from which a payroll capability computes
pay, so an error in the contract period or in the schedule selection eventually becomes an
error in a posted payslip.

### 2.5 The employee as an analytic and cost dimension

| Value supplied | Who consumes it |
|---|---|
| The Employee itself, as a link on expense records, timesheet lines, vehicle assignments and lunch orders | The [expenses](../expenses/README.md) domain, which posts expense reports; the [fleet](../fleet/README.md) domain, which attaches vehicle costs to a driver; the [lunch ordering](../lunch-ordering/README.md) domain, which maintains a wallet balance. |
| The Department, as a grouping dimension | Reporting throughout the platform. |

None of those links is itself an accounting fact; each becomes one only in the consuming
domain.

## 3. What a reimplementation must **not** do

- Do not post anything when a wage is entered or changed. The wage is documentation until a
  payroll capability turns it into a payslip.
- Do not post anything when a contract opens or closes. Closing a contract at departure has
  no accounting consequence in this domain.
- Do not create a payable when a bank account or a salary distribution is recorded.
- Do not derive the hourly cost from the wage, and do not derive the wage from the hourly
  cost. They are independent figures with different meanings — gross pay versus fully loaded
  internal cost.
- Do not attach a currency other than the employee's company currency to the wage or the
  hourly cost. Both are declared against the company currency and no conversion is performed
  anywhere in this domain.
