# Accounting effects

The Customer Relationship Management domain creates no journal entry, no journal item, no analytic entry and no valuation record. Nothing that happens to a Lead, a Pipeline Stage, a Sales Team, a scoring statistic or a reseller assignment touches the general ledger. This file states where the boundary lies, which amounts of this domain look financial but are not, and where the first real accounting effect appears.

## 1. Why there is no accounting effect

A Lead is an expectation, not a commitment. It records that somebody might buy something, with an estimated amount and an estimated chance. Generally accepted accounting principles recognize revenue when it is earned and measurable, which happens at the earliest when a contractual commitment exists and goods or services are delivered or transferred. An expectation held in a pipeline meets neither criterion, so no entry is posted and none must be posted by a replacement.

Consequently:

- marking an opportunity won posts nothing;
- marking an opportunity lost posts nothing and reverses nothing;
- merging two opportunities posts nothing;
- deleting a Lead posts nothing;
- changing the expected revenue, the recurring revenue or the probability posts nothing.

This is an industry-standard completion: no event of this domain produces a posting in the reference behaviour, and the accounting rationale above explains why a replacement must not add one.

## 2. The monetary amounts of this domain and what they are not

| Amount | What it is | What it is not |
|---|---|---|
| `expected_revenue` | A salesperson's estimate of the untaxed amount the customer would pay if the deal closed. | Not a receivable, not deferred revenue, not a contract asset, not a backlog figure. |
| `prorated_revenue` | The expected amount weighted by the probability, used for forecasting only. | Not an expected credit loss, not a provision, not an impairment. |
| `recurring_revenue` and its three derived figures | An estimate of a repeating amount over a stated number of months. | Not a contract liability, not a subscription schedule, not a revenue recognition plan. |
| `sale_amount_total` | A read-only sum of the untaxed amounts of the confirmed Sales Orders linked to the opportunity, converted into the company currency at the order date. | Not a ledger balance. It is computed on demand from the sales documents and is never stored as a posting. |
| `invoiced` and `invoiced_target` on a Sales Team | A read-only figure and a manual target used to draw a progress ratio on the team card. | Not a ledger balance; the figure is computed by the sales domain from the invoiced amounts of the period. |
| `turnover` in the Partner Assignment Analysis view | A read-only aggregation of the customer invoice analysis view, grouped per reseller. | Not a ledger account. The view reads invoice analysis rows; it never writes. |

None of these amounts is reconciled, none carries a currency rate that is fixed at posting time, and none appears in a trial balance.

## 3. Currency handling without accounting

The Lead carries a single derived currency, `company_currency`, equal to the currency of its company or, when the record has no company, of the company active in the session. All monetary fields of the Lead are expressed in that currency and no conversion ever happens inside the domain.

Two places read an amount expressed in another currency:

1. **The sum of confirmed orders.** Each order amount is converted from the order currency into the company currency of the opportunity, at the order date, in the company of the order. The conversion rule belongs to the general ledger domain; see [../general-ledger/calculations.md](../general-ledger/calculations.md). The result is a display figure, recomputed at every read.
2. **Raising the expected revenue from a confirmed order.** No conversion is attempted at all. When the currency of the order differs from the company currency of the opportunity, the expected revenue is left untouched (`LEAD-124`). This deliberately avoids storing a converted amount whose rate would silently age.

## 4. Analytic accounting

This domain writes no analytic distribution and no analytic line. A Lead has no analytic account and no analytic distribution field. When the opportunity becomes a quotation, the analytic distribution is determined by the sales domain from the customer, the products and the analytic distribution model; see [../analytic-accounting/README.md](../analytic-accounting/README.md).

## 5. Where the accounting boundary is crossed

The first record of this domain's chain that has an accounting consequence is the Sales Order, which belongs to the sales domain.

| Step | Domain | Accounting effect |
|---|---|---|
| A Lead is captured, qualified, converted, scored, assigned, merged, won or lost | Customer Relationship Management | none |
| A quotation is created from the opportunity | [Sales Management](../sales/README.md) | none while it is a quotation |
| The quotation is confirmed into a Sales Order | [Sales Management](../sales/README.md) | none in the general ledger; the order may create deliveries and may make the linked opportunity's expected revenue rise, which is still not an entry |
| The order is delivered | [Inventory Operations](../inventory-operations/README.md) and [Inventory Valuation and Costing](../inventory-valuation-and-costing/README.md) | the first entries appear here, when the valuation method requires them |
| A customer invoice is created from the order and posted | [Accounts Receivable and Customer Invoicing](../accounts-receivable/README.md) | the revenue and receivable entry is posted here |

A replacement therefore needs no ledger integration at all to implement this domain. It needs the general ledger only for two read-only purposes: the currency conversion of section 3, and the invoice analysis rows read by the Partner Assignment Analysis view.

## 6. The one adjacent posting rule to be aware of

The membership package of this domain lets a product grant a partner grade when a Sales Order containing it is confirmed (`LEAD-149`). Confirming that order is a sales operation and follows the sales domain's rules entirely; granting the grade is a pure master-data write on the customer's commercial entity, with no ledger consequence of its own. The revenue of the membership product is recognized exactly like the revenue of any other product on that order, by the sales and receivable domains.

## 7. Audit expectations

Although nothing is posted, the following traces must exist for an auditor reconstructing how a sale began:

| Trace | Where it lives |
|---|---|
| Who created the record and when | the shared audit fields `create_uid` and `create_date` |
| Every change of stage, salesperson, team, customer, contact data, expected revenue, recurring revenue, won status and lost reason | the discussion thread of the record, through the tracked fields listed in [entities.md](entities.md) |
| The time spent in each stage | the stage duration map of the record |
| Which records were merged into which, and what they contained | the merge summary note on the survivor, and the moved messages whose subject is prefixed with the name of the merged-away record |
| When the record was assigned and to whom | the assignment date, plus the tracked change of the salesperson |
| When the record was closed | the closing date, plus the tracked won or lost message |
| Which reseller received the record and what they answered | the tracked assigned partner, the forwarding date, the declined partner list and the posted acceptance or refusal message |

Deleting a Lead destroys these traces, which is why deletion is restricted to the sales administrator (`LEAD-155`) and why losing a deal archives rather than deletes.

## 8. Reconciliation notes

| Subject | The two statements | Resolution |
|---|---|---|
| Whether the domain posts anything | Both descriptions stated that it posts no journal entry, no journal item, no analytic entry and no valuation record. | Kept once, with the accounting rationale that explains why a rebuild must not add one, marked as the industry-standard completion it is. |
| The monetary amounts | One description listed the amounts and what they are not; the other only said the domain is not financial. | The table of section 2 is kept, because it is what stops a reader from mistaking an expectation for a receivable. |
| Where the boundary is crossed | Both agreed the first accounting effect is on the sales side. | The chain of section 5 is kept, naming the domain that carries each step. |
| The membership product | Only one description mentioned that a confirmed order may grant a partner level. | Section 6 states it and makes clear that granting a level is a master-data write with no ledger consequence of its own. |
