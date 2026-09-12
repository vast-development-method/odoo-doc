# Recruitment — Accounting Effects

The Recruitment domain produces **no journal entry, no journal item, no analytic amount, no
valuation layer and no payment**. Nothing in this domain debits or credits an account, and no
operation described in [workflows.md](workflows.md) reaches the general ledger.

This file states why that is the correct answer rather than an omission, draws the boundary
at which a financial consequence does begin, and names the domains that produce it. The
cross-domain hand-overs it describes are catalogued, with every other one, in
[cross-domain transactions](../cross-domain-transactions.md).

## 1. Why there is no ledger effect

1. **An application is not a commitment.** An Application records an intention on both sides:
   the person's intention to work for the organisation, and the organisation's intention to
   consider that person. Neither creates an obligation to pay, and neither changes an asset or
   a liability. Under accrual accounting no obligating event has occurred, so nothing is
   recognised.
2. **The salary fields are indicative.** The expected amount and the proposed amount are
   negotiation figures held for comparison and for reporting averages. They carry no currency
   field, they are never converted, they are never summed into a payable and they never reach
   an employment contract. They are privilege-restricted information, not accounting data.
   See [business-rules.md](business-rules.md#2-application-data-integrity), REC-015.
3. **The recruitment target is a headcount plan, not a budget.** The remaining target counts
   people, not money. Decrementing it when an Application reaches a hired stage changes a
   forecast of headcount and nothing else, as the worked example in
   [calculations.md](calculations.md#27-forecast-headcount) shows.
4. **Hiring hands over at the employee boundary.** Creating an Employee from an Application
   writes an Employee record and copies personal data, attachments and skills. The wage, the
   contract dates and the salary structure belong to the employment version owned by
   [Human Resources Core](../human-resources-core/README.md), and this domain writes none of
   them: the complete field mapping in
   [workflows.md](workflows.md#83-the-field-mapping-named-field-by-field) contains no wage, no
   contract date and no accounting field.
5. **Sourcing costs are not modelled here.** Job board subscriptions, advertising spend,
   referral bonuses and agency fees are purchases. They are recorded as vendor bills in
   [Accounts Payable](../accounts-payable/README.md). This domain stores only the attribution
   references — campaign, medium and source — that let those costs be attributed afterwards;
   it stores no amount for any of them.

## 2. Where the boundary lies, event by event

| Event in this domain | What is written here | Financial consequence, and where it is produced |
|---|---|---|
| An Application is created through any of the four channels | The Application, its attachments, its thread | none |
| An Application moves through the pipeline | The stage, the last-stage-update moment, the previous stage, the readiness colour | none |
| An Application enters a stage flagged as hired | The hire date; the position's remaining target is decremented | none |
| An Application leaves a hired stage | The hire date is cleared; the remaining target is incremented | none |
| An Application is refused or archived | The refusal reason, the refusal moment, the active flag | none |
| An Employee is created from an Application | The Employee, the two-way link, the copied attachments and skills | none in this domain. The employment cost begins when an employment version carrying a wage exists and payroll is run, in [Human Resources Core](../human-resources-core/README.md) and the payroll capability that consumes it |
| A written interview is sent or answered | The answer set and two thread entries | none |
| A text message is sent to a set of Applications | The messages in each thread | none in this domain. The consumption of the gateway's credits is billed by that gateway and recorded as a purchase in [Accounts Payable](../accounts-payable/README.md) |
| A Recruitment Source is created and used | The source, its Tracking Source, and the attribution references stamped on the applications it produces | none. Those references are analytical attributes used by reporting; they are **not** analytic accounting distributions, and no analytic line is created. See [Analytic Accounting](../analytic-accounting/README.md) for the mechanism this domain deliberately does not use |
| A Talent Pool is created, filled or emptied | The pool and the talents | none |
| A Job Position is created, published, archived or deleted | The position, its alias, its publication flags | none |

## 3. Reporting that looks financial and is not

The recruitment analysis screens aggregate the expected salary and the proposed salary as
**averages** — for example, the average expected salary per position, per source or per
department. These averages are decision aids for the recruiting team. They are not recognised
amounts, they are not reconciled against anything, they carry no currency, and a rebuild must
never feed them into the ledger.

The same caution applies to the two headcount figures. The remaining target and the forecast
headcount are planning numbers. An installation that wants a recruitment budget builds it in
[Financial Reporting](../financial-reporting/README.md) from purchase documents, not from
these fields.

## 4. What a rebuild must guarantee

1. No operation of this domain opens an accounting transaction. A rebuild may therefore
   implement the whole domain with no general ledger present at all, and the domain's tests
   must pass in that configuration.
2. Deleting or archiving any record of this domain leaves the ledger untouched. There is no
   entry to reverse, because there was never an entry to post.
3. The only cross-domain write that can eventually carry a financial consequence is the
   creation of an Employee, and that write is limited to the mapping in
   [workflows.md](workflows.md#83-the-field-mapping-named-field-by-field).
4. Multi-currency has no meaning inside this domain: the two salary amounts are unitless
   decimals by design (REC-015). A rebuild that adds a currency field is extending the
   specification, not implementing it.

## 5. Reconciliation notes

| Point | Resolution |
|---|---|
| Whether the domain has an accounting file at all | One draft supplied a reasoned statement of absence with a boundary table; the other referred the question to the hiring workflow. The reasoned statement is kept and extended with the per-event table of §2 and the guarantees of §4. |
| Whether the attribution references are analytic distributions | They are not. They are reporting attributes on the Application, and §2 states the distinction explicitly so that a rebuild does not wire them into analytic accounting. |
| Where sourcing costs live | Both drafts agree: vendor bills in the payables domain. §1 item 5. |
