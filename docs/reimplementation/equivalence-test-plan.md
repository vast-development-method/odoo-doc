# Equivalence test plan

How to prove that a rebuilt system behaves like this specification: eleven layers of test, the three fixed data sets they run on, forty-seven end-to-end golden scenarios with the records and the amounts they must produce, the invariants that must hold over the whole fixture at every moment, and the tolerance rules, which are: none for amounts, and documented rounding everywhere else.

The premise of the plan is that behavioral equivalence is not a judgement made at the end but a property held continuously, proved by a suite that grows alongside the rebuild. Each layer catches a class of divergence that the layers below it cannot see. A rebuild that passes the first five layers and has no sixth is not wrong; it is making a smaller claim, and [conformance profiles](conformance-profiles.md) names which claim.

---

## 1. What behavioral equivalence means here

Two systems are behaviorally equivalent when, for the same sequence of inputs on the same starting data, they produce:

1. **The same persistent records.** Same entities, same field values, same relations, same states. Identifiers may differ; everything a user or an integration can read must not.
2. **The same financial consequences.** Same journal entries, same items, same accounts, same debits and credits to the last unit of currency, same tax grids, same analytic attribution, same reconciliation links, same residual amounts.
3. **The same state transitions.** Same states reached, same transitions refused, same side effects at each transition.
4. **The same refusals.** Same operations refused, for the same reason, with the same message text, and with nothing written.
5. **The same computations.** Same numbers, including intermediate rounding, tie-breaking and the order in which amounts are summed.
6. **The same operational capabilities.** Same documents printed with the same content, same scheduled jobs with the same effect, same endpoints with the same request and response shapes, same exports.

Everything else may differ: the internal ordering of unordered results, surrogate identifiers, the timestamps of the run, the storage layout, the programming language.

### 1.1 The three kinds of difference and how each is treated

| Difference | Treatment |
|---|---|
| A number differs by any amount | Failure. There is no amount tolerance; section 19 states the rule. |
| A record is present in one system and absent in the other | Failure, including records a user never sees, when the specification says they exist (tracking values, reconciliation rows, valuation layers). |
| A field value differs only in an unordered relation's ordering | Not a failure, provided the specification declares no order for that relation. When it declares an order, an ordering difference is a failure. |

---

## 2. The eleven layers of test

Every layer states what it catches, where its cases come from, how it is built and what counts as passing. The order is the order in which a failure is cheapest to attribute: a rounding defect found in layer one costs minutes, and the same defect found through a failing golden scenario costs a day.

### Layer one: arithmetic and numeric precision

**What it catches.** Rounding drift, precision loss, a wrong evaluation order, the wrong currency at the moment of conversion, a tie broken the wrong way.

**Source of cases.** Every worked example in every `calculations.md` file, and every record of the mathematics catalogs under [`../../schemas/mathematics/`](../../schemas/mathematics/), whose worked examples carry inputs and exact expected outputs.

**Method.** Three families of case.

1. **Worked-example replication.** Every worked example becomes a test with its exact inputs and its exact expected outputs, including the intermediate values where the document shows them. Each case is a pure computation: no database, no records, no user. These tests must run in seconds, so that they can run on every change.
2. **Boundary sweeps.** For every formula that rounds, a sweep over values that land exactly on a tie (2.675, 0.005, −0.005, 1.005, 47.625), over zero, over the smallest representable amount, and over one step below and one step above a tie.
3. **Currency and unit variation.** Every monetary formula is run in a two-decimal currency, a zero-decimal currency, a currency whose step is 0.05 and a six-decimal currency; every quantity formula is run with a unit whose rounding step is 1, 0.01 and 0.001.

**Pass rule.** Exact equality of every produced number with the expected number, compared as decimals and never as binary floating point values.

**Coverage target.** Every formula in the specification has at least one case, and every formula with a rounding step has at least three: one that rounds down, one that rounds up and one that sits exactly on the boundary.

**Why first.** Arithmetic divergence is the most common and the most damaging failure in a financial rebuild, and it is the cheapest to catch. A tax computed a hundredth of a unit differently will not be noticed by a screen test but will fail an audit.

### Layer two: entity structure

**What it catches.** A missing field, a wrong type, a lost default, a dropped constraint, a renamed selection value, a relation with the wrong cardinality or the wrong deletion behavior.

**Source of cases.** [`../../schemas/data/entity-index.json`](../../schemas/data/entity-index.json), the per-entity documents under [`../../schemas/data/entities/`](../../schemas/data/entities/), [`../../schemas/data/relations.json`](../../schemas/data/relations.json), [`../../schemas/data/selection-values.json`](../../schemas/data/selection-values.json), [`../../schemas/data/constraints.json`](../../schemas/data/constraints.json), and, for a claim of record conformance, [`../../schemas/data/physical-tables.json`](../../schemas/data/physical-tables.json) and [`../../schemas/data/association-tables.json`](../../schemas/data/association-tables.json).

**Method.** Introspect the rebuild's own registry and schema and compare it, field by field, with the catalog. Report differences as three lists: missing, extra and divergent. Extra is not automatically a failure — a rebuild may add fields — but every entry must be acknowledged.

**Why here.** A structural gap makes every later test in that area meaningless, and it is found in seconds rather than through a business scenario failing hours later.

### Layer three: state machines

**What it catches.** A missing state, a transition that should be refused and is not, a guard evaluated on the wrong condition, a side effect that fires in the wrong order or not at all.

**Source of cases.** Every `state-machines.md`, and the catalog of state fields in [`../../schemas/data/state-machines.json`](../../schemas/data/state-machines.json).

**Method.** For each of the two hundred and fifty-two state fields of that catalog, build the transition table — from state, trigger, guard, to state, side effects — from the domain's `state-machines.md` and `workflows.md`, and generate:

1. One test per specified transition, asserting the destination state and every specified side effect: the records created, the fields changed, the messages posted and the activities scheduled.
2. One test per unspecified pair of states, asserting that the transition is refused with the message the domain declares, or that no operation exists that could attempt it.
3. One test per guard, in both its satisfied and its unsatisfied form.

**Shape of a case.**

```
entity:      Transfer
from:        assigned
trigger:     validate_transfer
guard:       every move line has a quantity and, for tracked products, a lot
to:          done
side effects: quantity records updated; valuation layers written; backorder offered when
              the done quantity is below the demand
refused when: the guard fails, with "You need to supply a Lot/Serial Number for product Coffee Beans."
```

**Pass rule.** Every transition of the state machine catalog is executed, and the count of refused transitions equals the count the catalog declares.

**Note.** The refusal half matters more than the permitted half. Most rebuilds implement the transitions that users perform and omit the guards that stop users performing the wrong one.

### Layer four: business scenarios

**What it catches.** A wrong interaction between rules that are individually correct, and a wrong ordering of side effects between domains.

**Source of cases.** Every workflow of every `workflows.md`, every numbered scenario of every `acceptance-criteria.md`, every trace in [cross-domain transactions](../domains/cross-domain-transactions.md), and the forty-seven golden scenarios of sections 5 to 17 of this document.

**Method.** Each case sets up the stated records, performs the stated operation as the actor the workflow names, and asserts every stated outcome: states, quantities, amounts, ledger items, reconciliations, statuses, the messages posted on the discussion thread and the activities scheduled. Scenarios run against a database seeded only as section 3 says, so that the preconditions of a scenario are entirely explicit.

**Shape of a case.**

```
workflow:    Confirm a sales order
actor:       a user of the Sales User access group
given:       quotation S00042 in state draft with two lines
when:        the Confirm operation is invoked
then:        the order state is sale
             the order date is the current moment
             the number is assigned from the sales sequence
             one Transfer is created in state assigned with two Stock Moves
             the discussion thread carries the message "Quotation confirmed"
             the customer receives no electronic mail
```

**Pass rule.** Every workflow of every domain is executed end to end at least once, and every branch the workflow describes — each decision point, each guard that can fail — is executed at least once.

**Why the cross-domain traces matter most.** A rebuild almost always gets a single-domain scenario right and a combined one wrong, because the combined one depends on ordering: whether the valuation layer is written before or after the ledger entry, whether the delivered quantity updates before the invoice status recomputes. The traces exist to pin that ordering down.

### Layer five: invariants

**What it catches.** Slow drift that no single scenario reveals.

**Method.** After every test of layers three, four, nine and ten, assert the fixed set of properties over the whole database that section 18 lists: that every posted entry balances per entry and per currency; that the balance of each valuation account equals the sum of the layers that feed it; that reconciled amounts net to zero within each reconciliation group in both currencies; that quantities are conserved per product and per location; that reserved quantity never exceeds the quantity on hand and never goes negative; that invoiced quantity never exceeds what the invoicing policy allows; that a record declaring a company is reachable only by a user whose company set contains it; and that every sequence has produced a strictly increasing series within each period, with gaps only where the specification permits them.

These checks are cheap and they fail loudly on exactly the errors that are hardest to attribute later.

### Layer six: contracts

**What it catches.** A route that moved, an argument that changed meaning, a response field that disappeared, an error that now reports a different kind, a structured document that no longer validates.

**Source of cases.** The [endpoint catalog](../interfaces/endpoint-catalog.md), the [remote transport contracts](../interfaces/remote-transport-contracts.md), the [service layer](../interfaces/service-layer.md), the [external integrations](../interfaces/external-integrations.md) and [`../../schemas/interfaces/routes.json`](../../schemas/interfaces/routes.json).

**Method.** For each route: call it with a valid request, with each declared parameter missing, with a malformed parameter, with an unauthenticated session, with a session lacking the required group, and with a repeated request where repetition matters. Compare the response shape, the status semantics, the error envelope and the side effects. Replay a recorded set of exchanges against the rebuild and compare responses field by field, ignoring only the fields this plan marks as varying. Round-trip every structured document format: encode a document, decode it, and assert that the decoded record matches the original.

For each external connector — payment providers, shipping carriers, payment terminals, exchange networks, calendar providers, mail servers — drive it against a simulated counterpart that produces, in turn, every documented response including the failure and the timeout, and assert the state the connector reaches and the records it writes. A connector must never leave a document in an undefined state after a timeout: the post-processing job must be able to resolve it.

**Pass rule.** Every route is exercised in its success form and in every documented failure form; every connector reaches every documented state.

**Required only for a claim of contract conformance**, but valuable earlier as a regression net on anything a client already consumes.

### Layer seven: authorization

**What it catches.** A record visible to someone who should not see it, an operation permitted to a group that should be refused, a field readable through a relation that is restricted directly.

**Source of cases.** [`../../schemas/operational/groups.json`](../../schemas/operational/groups.json), [`../../schemas/operational/access-rights.json`](../../schemas/operational/access-rights.json), [`../../schemas/operational/record-rules.json`](../../schemas/operational/record-rules.json) and the [access matrix by group](../references/access-matrix-by-group.md).

**Method.** Two matrices and a set of always-tested cases.

1. **The entity matrix.** For each group, construct a user in that group and only that group, and for each entity attempt create, read, update and delete on a record of the fixture, comparing the outcome with the shipped access rule. The matrix has one row per entity and group pair.
2. **The record matrix.** For each record rule, build one record the rule admits and one it excludes, and assert that a user of the rule's group reads the first and not the second — in a query, in a grouped read, in a report and through every endpoint that exposes the entity.

Always tested as well: that a field restricted to a group is absent from the read result of a user outside it and refused on write; that a user whose allowed companies are two of three reads the records of those two and not of the third, and that a document mixing companies is refused; that a portal user reaching another customer's document receives the refusal and never the record; that a public visitor reaching an unpublished record receives the not-found behavior and never the record; that a restricted record cannot be reached indirectly by following a relation from a permitted one; and that a scheduled job runs with its declared identity and is subject to that identity's record rules unless the job declares otherwise.

**Pass rule.** The produced matrices equal the shipped matrices cell by cell.

**Why a separate layer.** Authorization failures are silent. A business scenario passes whether or not the data was protected, because the scenario runs as a privileged user.

### Layer eight: concurrency and recovery

**What it catches.** Lost updates, two workers running one scheduled job, a reservation granted twice, a partially applied operation surviving a failure.

**Method.** Drive conflicting operations in parallel and assert the specified resolution: two operators reserving the last unit, two closing the same session, two posting into the same numbering sequence, two reconciling the same item, two counting the same quantity record. Then interrupt operations at defined points and assert that the recovered state contains no partial effect, and that an operation retried after a serialization failure produces exactly one effect.

**Required only for a claim of operational conformance.**

### Layer nine: business rules

**What it catches.** A validation that does not fire, a default that is not applied, a derivation that reads the wrong input, a constraint that is missing, an on-change behavior that writes a different value.

**Source of cases.** Every numbered rule of every `business-rules.md`, and the message index in [validation messages](../references/validation-messages.md).

**Method.** One test per rule identifier. The test names the rule identifier, sets up the smallest record set that can trigger it, performs the action, and asserts either the written value or the exact refusal message. A rule with branches — a setting that changes the behavior — has one test per branch.

**Shape of a case.**

```
rule:        BOOK-RULE-001
given:       an Account with the code 400000 in the reference company
when:        a second Account with the code 400000 is created in the same company
then:        the creation is refused with
             "Account codes must be unique. You can't create accounts with these duplicate codes: 400000"
and:         no Account record was written
```

**Pass rule.** Every numbered rule of every domain has at least one test, and every one passes.

**Coverage measurement.** The coverage matrix of [traceability rules](traceability-rules.md) lists every rule identifier and the tests that cover it; an uncovered rule is a build blocker, not a warning.

### Layer ten: accounting consequences

**What it catches.** An entry posted on the wrong account, a side chosen wrongly, a missing item, an amount that does not follow the account selection precedence, a tax grid that is not stamped, an analytic distribution that is not written.

**Source of cases.** Every triggering event of every `accounting-effects.md`.

**Method.** One test per triggering event. The test performs the event and compares the full item listing: account code, account name, partner, label, debit, credit, currency and currency amount, tax grid stamps, analytic distribution and reconciliation state.

**Shape of a case.**

```
event:       Post a customer invoice of 996.00 excluding tax with a 21 percent tax
then the entry contains exactly:
| Account                  | Partner   | Label           | Debit   | Credit  |
| 121000 Trade Receivables | Deltatech | INV/2026/00001  | 1205.16 |         |
| 400000 Product Sales     |           | Office Desk     |         |  600.00 |
| 400000 Product Sales     |           | Office Chair    |         |  396.00 |
| 251000 Tax Received      |           | Sales Tax 21%   |         |  209.16 |
and:         the sum of debits equals the sum of credits
and:         the base lines carry the grid of sales at twenty-one percent
```

**Pass rule.** Every event listed in every `accounting-effects.md` is tested, including the reversal of each event and the effect of each account-selection fallback: the product, then the product category, then the company default.

### Layer eleven: report content

**What it catches.** A printed or exported document that loses a column, a subtotal, a tax summary, a payment communication or a page number, or that groups its lines differently.

**Source of cases.** Every report of [`../../schemas/interfaces/report-actions.json`](../../schemas/interfaces/report-actions.json), with the content lists in the domains' `interfaces.md` files and in [report and export documents](../interfaces/report-and-export-documents.md).

**Method.** Render each report for a fixture record and assert, field by field, the content list of the domain: the header block, the addresses, the document number and dates, the line table with its columns, the subtotals per section, the tax summary, the total, the payment communication, the terms, the signature block and the page numbering. Exports are compared as data: same columns, same order, same values, same number formatting rules.

**Pass rule.** Every report is rendered for at least one record of each variant it supports — with and without taxes, with and without discounts, in a foreign currency, with sections, over several pages — and every asserted field matches.

---

## 3. The fixture data sets

Three fixed data sets are versioned alongside the rebuild, and every test names the one it runs on.

**The reference set.** The shipped reference data only: countries, country groups, currencies, languages, units of measure, decimal precisions, activity types, message subtypes and the other records that the seed data tables of the [build sequence](build-sequence.md) list per step. Every layer one, two, three and nine test starts here and creates what it needs, which keeps preconditions explicit and failures reproducible.

**The configured set.** The reference set plus one configured business: the reference company of this section, with its chart of accounts, journals, taxes, warehouse, locations, operation types, pricelists, employees, counter and a product for each costing method and each tracking mode. The golden scenarios and the cross-domain traces run on it.

**The migration set.** For a claim of record conformance or higher in [conformance profiles](conformance-profiles.md), an extract of real records with every field populated at least once, used to prove that a load leaves nothing unmapped and no value reinterpreted.

No test depends on demonstration data, and no test depends on another test's leftovers.

Every test in this plan runs on the configured set described here. The fixture is built once, by loading the shipped records of the build sequence and then creating the records below in the order given. Nothing in the fixture depends on the current date except where a scenario says so; all scenarios use the fictional year 2026.

### 3.1 Companies and currencies

| Company | Currency | Country | Fiscal year end | Rounding of the currency |
|---|---|---|---|---|
| Northwind Trading | euro | Belgium | 31 December | step 0.01, two decimal places |
| Northwind Services | euro | Belgium | 31 December | step 0.01, two decimal places |
| Northwind Pacific | United States dollar | United States | 31 December | step 0.01, two decimal places |

Northwind Trading and Northwind Services are sister companies under the same parent for multi-company tests; Northwind Pacific is a separate root company whose currency differs from the other two.

Currency rates, expressed as units of the foreign currency for one euro:

| Date | United States dollar | pound sterling | Japanese yen |
|---|---:|---:|---:|
| 2026-01-01 | 1.1000 | 0.8500 | 160.00 |
| 2026-02-01 | 1.2500 | 0.8600 | 162.00 |
| 2026-03-01 | 1.0000 | 0.8400 | 158.00 |

The Japanese yen has zero decimal places and a rounding step of 1; the other currencies have two decimal places and a step of 0.01. A fourth currency, "Test Five Cents", is defined as a copy of the euro with a rounding step of 0.05 and is used only by the precision tests. A fifth, "Test Six Decimals", has a step of 0.000001.

### 3.2 Chart of accounts of Northwind Trading

| Code | Name | Type | Reconcilable |
|---|---|---|---|
| 101000 | Cash on Hand | asset, cash | no |
| 101100 | Bank Current Account | asset, cash | no |
| 101110 | Bank Account in United States Dollars | asset, cash | no |
| 101200 | Outstanding Receipts | asset, current | yes |
| 101300 | Outstanding Payments | asset, current | yes |
| 101400 | Inter-Bank Transfer Account | asset, current | yes |
| 110100 | Stock Interim Received | asset, current | yes |
| 110200 | Stock Interim Delivered | asset, current | yes |
| 110300 | Stock Valuation | asset, current | no |
| 110400 | Production (Work in Progress) | asset, current | no |
| 121000 | Trade Receivables | asset, receivable | yes |
| 131000 | Tax Paid | asset, current | no |
| 151000 | Prepaid Expenses | asset, current | no |
| 211000 | Trade Payables | liability, payable | yes |
| 211100 | Down Payments Received | liability, current | no |
| 211200 | Gift Card Liability | liability, current | no |
| 251000 | Tax Received | liability, current | no |
| 261000 | Employee Payables | liability, payable | yes |
| 271000 | Point of Sale Receivable | asset, receivable | yes |
| 301000 | Share Capital | equity | no |
| 321000 | Retained Earnings | equity | no |
| 999999 | Undistributed Result | equity, unallocated | no |
| 400000 | Product Sales | income | no |
| 400100 | Service Sales | income | no |
| 400200 | Sales Discounts | income | no |
| 400300 | Export and Intra-Community Sales | income | no |
| 410000 | Other Income | income, other | no |
| 411000 | Foreign Exchange Gain | income, other | no |
| 412000 | Early Payment Discount Obtained | income, other | no |
| 500000 | Cost of Goods Sold | expense | no |
| 500100 | Purchase Price Difference | expense | no |
| 510000 | Inventory Variation | expense | no |
| 600000 | Operating Expenses | expense | no |
| 600100 | Travel Expenses | expense | no |
| 600200 | Meal Expenses | expense | no |
| 610000 | Payroll Expenses | expense | no |
| 610100 | Work Center Expenses | expense | no |
| 620000 | Cash Difference Loss | expense | no |
| 620100 | Cash Difference Gain | income, other | no |
| 620200 | Cash Rounding | expense | no |
| 630000 | Foreign Exchange Loss | expense, other | no |
| 640000 | Early Payment Discount Granted | expense, other | no |

Company defaults: income account 400000, expense account 600000, price difference account 500100, exchange gain 411000, exchange loss 630000, early discount granted 640000, early discount obtained 412000, cash difference gain 620100, cash difference loss 620000, transfer account 101400, point of sale receivable 271000.

**Which locations carry a valuation account.** This decides where an entry is posted at movement time and where it is deferred. In the fixture:

| Location | Valuation account | Consequence |
|---|---|---|
| every internal location (`MW/...`, `SD/...`) | none | internal moves post nothing |
| `Vendors` (partner location) | none | a receipt posts nothing; the stock valuation account is debited when the vendor bill is posted, and the receipt is re-valued to the billed amount |
| `Customers` (partner location) | none | a delivery posts nothing; the cost of goods sold is recognized when the customer invoice is posted |
| `Production` (virtual, production usage) | 110400 Production (Work in Progress) | component consumption and finished production post their two-line entries, and the account nets to zero for a completed order |
| `Inventory adjustment` (virtual) | 510000 Inventory Variation | a count difference posts at validation |
| `Scrap` (virtual) | 510000 Inventory Variation | a scrap posts at validation |

The accounts 110100 Stock Interim Received and 110200 Stock Interim Delivered exist in the chart and are used only by the variant runs of section 8.4, which repeat three scenarios with two-step receipt and delivery locations that carry them.

### 3.3 Journals of Northwind Trading

| Name | Code | Type | Default account | Numbering prefix |
|---|---|---|---|---|
| Customer Invoices | INV | sale | 400000 | `INV/%(year)s/` |
| Vendor Bills | BILL | purchase | 600000 | `BILL/%(year)s/` |
| Bank | BNK1 | bank | 101100 | `BNK1/%(year)s/` |
| Cash | CSH1 | cash | 101000 | `CSH1/%(year)s/` |
| Miscellaneous | MISC | general | none | `MISC/%(year)s/` |
| Inventory Valuation | STCK | general | none | `STCK/%(year)s/` |
| Point of Sale | PSAL | general | none | `PSAL/%(year)s/` |
| Exchange Difference | EXCH | general | none | `EXCH/%(year)s/` |

The bank journal has outstanding receipts 101200, outstanding payments 101300, profit account 620100, loss account 620000 and a suspense account 101400.

### 3.4 Taxes

| Name | Kind | Rate or amount | Price included | Accounts | Grids on base / on tax |
|---|---|---:|---|---|---|
| Sales Tax 21% | percentage | 21 | no | 251000 | `+03` / `+54` |
| Sales Tax 6% | percentage | 6 | no | 251000 | `+01` / `+54` |
| Sales Tax 21% Included | percentage | 21 | yes | 251000 | `+03` / `+54` |
| Purchase Tax 21% | percentage | 21 | no | 131000 | `+81` / `+59` |
| Purchase Tax 6% | percentage | 6 | no | 131000 | `+81` / `+59` |
| Zero-Rated Export | percentage | 0 | no | none | `+47` / none |
| Intra-Community Sales 0% | percentage | 0 | no | none | `+46` / none |
| Intra-Community Purchase 21% | group of two | +21 and −21 | no | 131000 and 251000 | `+86` / `+59` and `+56` |
| Environmental Contribution | fixed per unit | 0.50 | no | 251000 | `+03` / `+54` |
| Sales Tax 6% Included | percentage | 6 | yes | 251000 | `+01` / `+54` |
| Withholding on Services 15% | percentage | −15 | no | 211000 | none / `+80` |
| Cash Basis Service Tax 21% | percentage, cash basis | 21 | no | 251000 through 151000 | `+03` / `+54` |

Tax groups: Standard Rate (21 percent taxes), Reduced Rate (6 percent taxes), Zero Rate, Withholding.

### 3.5 Fiscal positions

| Name | Detection | Tax mapping | Account mapping |
|---|---|---|---|
| Domestic | default, no automatic detection | none | none |
| Intra-Community | country group "European Union" excluding Belgium, customer has a tax identification number | Sales Tax 21% to Intra-Community Sales 0%; Sales Tax 6% to Intra-Community Sales 0%; Purchase Tax 21% to Intra-Community Purchase 21% | 400000 to 400300 |
| Export | every country outside the country group "European Union" | Sales Tax 21% and 6% to Zero-Rated Export | 400000 to 400300 |
| Cash Basis | manual | Sales Tax 21% to Cash Basis Service Tax 21% | none |
| Counter Tax Included | set on the counter, not detected | Sales Tax 21% to Sales Tax 21% Included; Sales Tax 6% to Sales Tax 6% Included | none |

### 3.6 Payment terms

| Name | Installments | Early discount |
|---|---|---|
| Immediate | 100 percent at 0 days | none |
| 30 Days | 100 percent at 30 days | none |
| 30 Days End of Month | 100 percent at 30 days, end of month | none |
| 2/10 Net 30 | 100 percent at 30 days | 2 percent within 10 days, mode "On early payment" |
| 2/10 Net 30 Reduced Base | 100 percent at 30 days | 2 percent within 10 days, mode "Always (upon invoice)" |
| 2/10 Net 30 Full Base | 100 percent at 30 days | 2 percent within 10 days, mode "Never" |
| 30 Now, Balance 30 Days | 30 percent at 0 days, 70 percent at 30 days | none |
| Three Equal Thirds | three installments of 33.34, 33.33 and 33.33 percent at 0, 30 and 60 days | none |

### 3.7 Units of measure and packagings

| Category | Unit | Kind | Ratio to the reference | Rounding step |
|---|---|---|---:|---:|
| Unit | Units | reference | 1 | 0.01 |
| Unit | Dozen | bigger | 12 | 0.01 |
| Unit | Pack of Six | bigger | 6 | 1 |
| Weight | Kilogram | reference | 1 | 0.001 |
| Weight | Gram | smaller | 0.001 | 1 |
| Weight | Tonne | bigger | 1000 | 0.001 |
| Volume | Litre | reference | 1 | 0.01 |
| Working Time | Hours | reference | 1 | 0.01 |
| Working Time | Days | bigger | 8 | 0.01 |

Packagings: "Box of Twelve" holding 12 Units of Paper Ream; "Pallet of Forty Boxes" holding 480 Units of Paper Ream; "Sack of Twenty-Five Kilograms" holding 25 Kilograms of Coffee Beans.

### 3.8 Products

| Name | Kind | Stock unit | Costing | Cost | Sales price | Sales tax | Income account | Expense account | Tracking |
|---|---|---|---|---:|---:|---|---|---|---|
| Office Desk | goods, storable | Units | average | 120.00 | 300.00 | Sales Tax 21% | 400000 | 500000 | none |
| Office Chair | goods, storable | Units | first in first out | 45.00 | 99.00 | Sales Tax 21% | 400000 | 500000 | none |
| Desk Lamp | goods, storable | Units | standard | 8.00 | 19.99 | Sales Tax 21% | 400000 | 500000 | none |
| Paper Ream | goods, storable | Units | average | 3.50 | 6.00 | Sales Tax 6% | 400000 | 500000 | none |
| Coffee Beans | goods, storable | Kilogram | first in first out | 12.00 | 25.00 | Sales Tax 6% | 400000 | 500000 | by lot, with expiration |
| Table Top | goods, storable | Units | average | 60.00 | 0.00 | none | 400000 | 500000 | none |
| Table Leg | goods, storable | Units | average | 7.50 | 0.00 | none | 400000 | 500000 | none |
| Dining Table | goods, storable, manufactured | Units | average | derived | 480.00 | Sales Tax 21% | 400000 | 500000 | by serial number |
| Desk Set | goods, kit | Units | not valued | derived | 399.00 | Sales Tax 21% | 400000 | 500000 | none |
| Consulting Hour | service | Hours | none | 60.00 | 120.00 | Sales Tax 21% | 400100 | 600000 | none |
| Installation Service | service | Units | none | 0.00 | 250.00 | Sales Tax 21% | 400100 | 600000 | none |
| Delivery Charge | service | Units | none | 0.00 | 0.00 | Sales Tax 21% | 400100 | 600000 | none |
| Down Payment | service | Units | none | 0.00 | 0.00 | from the order | 121000 as a prepayment | none | none |
| Gift Card | service | Units | none | 0.00 | 0.00 | none | 400100 | 600000 | none |
| Event Seat | service, event ticket | Units | none | 0.00 | 150.00 | Sales Tax 21% | 400100 | 600000 | none |

Product categories:

| Category | Costing | Valuation | Stock valuation account | Expense account | Production account | Price difference account | Removal strategy | Products |
|---|---|---|---|---|---|---|---|---|
| All / Furniture | average cost | automated | 110300 | 500000 | 110400 | 500100 | first in first out | Office Desk, Office Chair, Table Top, Table Leg, Dining Table |
| All / Supplies | average cost | automated | 110300 | 500000 | 110400 | 500100 | first in first out | Paper Ream |
| All / Fixtures | standard price | automated | 110300 | 500000 | 110400 | 500100 | first in first out | Desk Lamp |
| All / Beverages | first in first out | automated | 110300 | 500000 | 110400 | 500100 | first expired first out | Coffee Beans |
| All / Services | none | none | none | 600000 | none | none | none | every service product |

Two extra products complete the fixture: "Freight Landed Cost", a service product of kind landed cost with the expense account 600000 and the split method "by value"; and "Standard Delivery", a service product carrying Sales Tax 21% used as the delivery line of the "Standard Delivery" method.

Bills of materials:

| Finished product | Kind | Quantity produced | Components | Operations |
|---|---|---:|---|---|
| Dining Table | manufacture | 1 | 1 Table Top, 4 Table Legs | Assembly on work center "Assembly Line" for 30 minutes |
| Desk Set | kit | 1 | 1 Office Desk, 1 Office Chair, 1 Desk Lamp | none |

Work centers: "Assembly Line", capacity one unit at a time, hourly cost 45.00, expense account 610100, working schedule "Standard 40 Hours", time efficiency 100 percent, no setup or cleanup time.

Vendor prices: Table Top from Baltic Timber at 60.00 for a minimum of 10, lead time 5 days; Table Leg from Baltic Timber at 7.50 for a minimum of 40, lead time 5 days; Desk Lamp from Lumina Lighting at 8.00, lead time 3 days; Office Chair from Baltic Timber at 45.00, lead time 7 days.

### 3.9 Pricelists

| Name | Currency | Rules |
|---|---|---|
| Standard | euro | one global rule: the product sales price, no discount |
| Wholesale | euro | rule 1: on the category "All / Furniture", 10 percent off the sales price for a minimum quantity of 10; rule 2 (global fallback): the sales price |
| Export | United States dollar | one global rule: the sales price converted at the rate of the document date, rounded to 0.01, surcharge 0.00 |
| Promotional | euro | rule 1: on the product Desk Lamp, a fixed price of 15.00 from 2026-03-01 to 2026-03-31; rule 2 (global fallback): the sales price |

### 3.10 Partners

| Name | Role | Country | Fiscal position | Payment terms | Pricelist | Currency |
|---|---|---|---|---|---|---|
| Deltatech Solutions | customer | Belgium | Domestic | 30 Days | Standard | euro |
| Meridian Bakery | customer | Belgium | Domestic | Immediate | Standard | euro |
| Atlas Imports | customer | Netherlands, with a tax identification number | Intra-Community | 30 Days | Standard | euro |
| Pacific Retail | customer | United States | Export | 30 Days | Export | United States dollar |
| Harbour Group | customer | Belgium | Domestic | 2/10 Net 30 | Standard | euro |
| Baltic Timber | vendor | Belgium | Domestic | 30 Days | none | euro |
| Pacific Supplies | vendor | United States | Export | 30 Days | none | United States dollar |
| Lumina Lighting | vendor | Belgium | Domestic | Immediate | none | euro |
| Nordic Components | vendor | Sweden, with a tax identification number | Intra-Community | 30 Days | none | euro |
| Crescent Logistics | carrier and vendor | Belgium | Domestic | 30 Days | none | euro |
| Walk-in Customer | counter customer | Belgium | Domestic | Immediate | Standard | euro |

### 3.11 Warehouses, locations and delivery methods

**Main Warehouse**, short code `MW`, one-step receipt and one-step delivery by default. Locations: `MW/Stock` with children `MW/Stock/Shelf A` and `MW/Stock/Shelf B`, `MW/Input`, `MW/Quality Control`, `MW/Packing`, `MW/Output`. Partner locations: `Vendors`, `Customers`. Virtual locations: `Inventory adjustment`, `Scrap`, `Production`.

**South Depot**, short code `SD`, two-step delivery (pick then ship), resupplied from Main Warehouse.

Storage category "Heavy Shelf" with a maximum of 20 Units, applied to `MW/Stock/Shelf B`, with a putaway rule sending Office Desk to Shelf B.

Delivery methods: "Standard Delivery" at a fixed 15.00 with free delivery above 1 000.00; "Weight-Based Delivery" at 5.00 plus 0.75 per kilogram; "Pickup in Store" at 0.00. All three invoice through the "Standard Delivery" service product and carry Sales Tax 21%.

Reordering rules: Table Leg, minimum 40, maximum 120, multiple of 20, route Buy, vendor Baltic Timber; Desk Lamp, minimum 50, maximum 200, multiple of 1, route Buy, vendor Lumina Lighting.

Routes beyond the shipped ones: "Dropship" on Desk Lamp, making a confirmed sales order line for that product buy from Lumina Lighting and ship straight to the customer.

### 3.12 Employees, schedules and the counter

| Employee | Department | Schedule | Hourly cost | Time off allocation |
|---|---|---|---:|---|
| Anna Lindqvist | Sales | Standard 40 Hours, Monday to Friday, 08:00 to 12:00 and 13:00 to 17:00 | 35.00 | 20 days paid time off for 2026 |
| Bruno Ferraro | Warehouse | Standard 40 Hours | 28.00 | 20 days paid time off for 2026 |
| Clara Mensah | Consulting | Standard 40 Hours | 60.00 | 20 days paid time off for 2026, accrual plan "Two Days a Month" |

Public holidays for 2026: 1 January, 1 May, 21 July, 25 December.

Counter "Front Shop": journal PSAL, invoice journal INV, fiscal position "Counter Tax Included" so that displayed prices include tax, cash payment method on journal CSH1 with account 101000, card payment method on journal BNK1 with outstanding account 101200, customer account payment method posting to 271000, opening cash balance 200.00, cash rounding to 0.05 applied to cash payments only with the difference on 620200, products Office Desk, Office Chair, Desk Lamp, Paper Ream, Coffee Beans and Event Seat available at the counter, cash denominations of 0.05, 0.10, 0.20, 0.50, 1, 2, 5, 10, 20, 50, 100.

Restaurant extension of the same counter for the restaurant scenarios: floor "Main Room" with tables 1 to 6, table 5 seating four.

Loyalty and promotion programs: "Spring Promotion", a promotion granting ten percent off the order when the untaxed amount reaches 500.00, valid 2026-03-01 to 2026-05-31, on the counter and the storefront; "WELCOME10", a coupon program granting ten percent off the cart, one use per coupon, code `WELCOME10`; "Gift Card", a gift card program selling the Gift Card product and crediting 211200.

Event "Spring Product Days" on 2026-04-15 with the ticket "Standard Seat" at 150.00 carrying Sales Tax 21%, one hundred seats, sold through the Event Seat product; event "Open House" on 2026-05-20 with a free ticket and unlimited seats.

### 3.13 Opening stock and opening balances

Loaded by an inventory adjustment dated 2026-01-01 and an opening entry dated 2026-01-01.

| Product | Location | Quantity | Unit cost | Value |
|---|---|---:|---:|---:|
| Office Desk | MW/Stock/Shelf B | 20 | 120.00 | 2 400.00 |
| Office Chair | MW/Stock/Shelf A | 40 | 45.00 | 1 800.00 |
| Desk Lamp | MW/Stock/Shelf A | 100 | 8.00 | 800.00 |
| Paper Ream | MW/Stock/Shelf A | 240 | 3.50 | 840.00 |
| Coffee Beans, lot `L-2026-A` expiring 2026-06-30 | MW/Stock/Shelf A | 50 | 12.00 | 600.00 |
| Table Top | MW/Stock | 10 | 60.00 | 600.00 |
| Table Leg | MW/Stock | 40 | 7.50 | 300.00 |

Total opening stock value: 7 340.00 on account 110300.

Opening entry: debit 110300 Stock Valuation 7 340.00, debit 101100 Bank Current Account 50 000.00, credit 301000 Share Capital 57 340.00.

### 3.14 Fixture invariants

After loading, the fixture satisfies:

1. The trial balance is balanced and the only non-zero accounts are 110300 (7 340.00 debit), 101100 (50 000.00 debit) and 301000 (57 340.00 credit).
2. The sum of the valuation layers equals 7 340.00 and equals the balance of 110300.
3. No document is in a draft state, no transfer is open, no reservation exists.
4. Every shipped record of the build sequence's seed tables exists with its external identifier.
5. The fixture loads in under one run of the loader, with no manual step and no dependency on the current date.

---

## 4. Golden scenarios: how to read them

Each scenario gives its purpose, its preconditions beyond the fixture, its numbered steps, and the expected records with their amounts. Amounts are exact. Where a scenario shows a journal entry, the table is the complete entry: no item may be missing and no item may be added.

Scenarios are numbered `GS-01` to `GS-47` and are cited by the gates of [milestones](milestones.md).

| Family | Scenarios |
|---|---|
| Quote to cash | GS-01 to GS-08 |
| Procure to pay | GS-09 to GS-14 |
| Replenishment and routes | GS-15 to GS-17 |
| Inventory operations | GS-18 to GS-21 |
| Make to stock | GS-22 to GS-26 |
| A point of sale day | GS-27 to GS-30 |
| Hire to retire | GS-31 to GS-34 |
| Expense to reimbursement | GS-35 to GS-36 |
| Lead to opportunity to order | GS-37 to GS-38 |
| Event registration to invoice | GS-39 to GS-40 |
| Storefront checkout and portal | GS-41 to GS-43 |
| Projects and timesheets | GS-44 to GS-45 |
| Period close | GS-46 to GS-47 |

**Starting point.** Every scenario starts from the freshly loaded fixture of section 3 unless it names a predecessor scenario, in which case it starts from the state that predecessor leaves. Every date is in 2026. Every amount is in euro unless the scenario says otherwise.

---

## 5. Golden scenarios, quote to cash

### GS-01: Quotation, order, delivery, invoice, payment

**Purpose.** The complete sale of goods with the invoicing policy "delivered quantities", proving order creation, reservation, delivery, invoice posting, cost of goods sold recognition and payment reconciliation.

**Steps and expected results.**

1. On 2026-02-02 a salesperson creates a quotation for Deltatech Solutions. The quotation takes the pricelist Standard, the payment terms 30 Days, the fiscal position Domestic and the delivery and invoice addresses of the customer. State `draft`, no number.
2. Two lines are added: 2 Office Desk and 4 Office Chair. The unit prices come from the pricelist: 300.00 and 99.00. Line subtotals 600.00 and 396.00. Untaxed amount 996.00, tax 209.16, total 1 205.16.

```formula
tax = round_to(996.00 × 0.21, 0.01) = round_to(209.16, 0.01) = 209.16
```

3. The quotation is confirmed. The state becomes `sale`, the number `S00001` is assigned, the order date is 2026-02-02, and one Transfer `MW/OUT/00001` is created with two Stock Moves, both reserved from the opening stock, scheduled for 2026-02-02.
4. The delivery is validated on 2026-02-03. Stock on hand becomes 18 Office Desk and 36 Office Chair. Valuation layers: −240.00 for the desks (2 at the average cost 120.00) and −180.00 for the chairs (4 consumed from the opening layer at 45.00). **No journal entry is posted**, because the destination location `Customers` carries no valuation account.
5. An invoice is created from the order. Because the invoicing policy is "delivered quantities" and the delivery is complete, both lines are proposed in full. The invoice is posted on 2026-02-03 with the number `INV/2026/00001` and the due date 2026-03-05.

| Account | Partner | Label | Debit | Credit |
|---|---|---|---:|---:|
| 121000 Trade Receivables | Deltatech Solutions | INV/2026/00001 | 1 205.16 | |
| 400000 Product Sales | | Office Desk | | 600.00 |
| 400000 Product Sales | | Office Chair | | 396.00 |
| 251000 Tax Received | | Sales Tax 21% | | 209.16 |
| 500000 Cost of Goods Sold | | Office Desk | 240.00 | |
| 500000 Cost of Goods Sold | | Office Chair | 180.00 | |
| 110300 Stock Valuation | | Office Desk | | 240.00 |
| 110300 Stock Valuation | | Office Chair | | 180.00 |

Debits 1 625.16, credits 1 625.16. The two base lines carry the grid `+03`, the tax line the grid `+54`.

6. On 2026-03-04 a payment of 1 205.16 is registered on the bank journal. The payment entry debits 101200 Outstanding Receipts 1 205.16 and credits 121000 Trade Receivables 1 205.16, and is reconciled with the invoice's receivable item. The invoice's payment state becomes `paid`, its residual 0.00.
7. The bank statement line of 2026-03-05 is imported and reconciled with the payment: debit 101100 Bank Current Account 1 205.16, credit 101200 Outstanding Receipts 1 205.16.

**Final balances touched by the scenario:** 101100 debit 1 205.16, 121000 zero, 251000 credit 209.16, 400000 credit 996.00, 500000 debit 420.00, 110300 credit 420.00.

### GS-02: Quotation template with an optional line

**Purpose.** Templates, sections, notes and optional lines.

1. A quotation for Deltatech Solutions is created and the template "Office Fit-Out" is applied. The lines become: a section "Furniture", 4 Office Desk at 300.00, 8 Office Chair at 99.00, a note "Delivery within two weeks", and one optional line for 1 Installation Service at 250.00.
2. The section and the note contribute nothing to the totals. Untaxed 1 992.00, tax 418.32, total 2 410.32.
3. The customer accepts the option in the portal. The optional line becomes an ordinary line. Untaxed 2 242.00, tax 470.82, total 2 712.82.

```formula
1992.00 × 0.21 = 418.32
2242.00 × 0.21 = 470.82
```

4. Confirming creates a delivery for the two goods lines only; the service line creates no move.

### GS-03: Down payment of thirty percent, then the final invoice

**Purpose.** Down payment invoicing, its account and its deduction.

1. An order for Deltatech Solutions with 4 Office Desk at 300.00 is confirmed. Untaxed 1 200.00, tax 252.00, total 1 452.00.
2. A down payment invoice of 30 percent is created on the untaxed amount. The invoice has one line on the Down Payment product, amount 360.00, carrying the tax of the order lines.

| Account | Debit | Credit |
|---|---:|---:|
| 121000 Trade Receivables | 435.60 | |
| 211100 Down Payments Received | | 360.00 |
| 251000 Tax Received | | 75.60 |

3. The delivery of the four desks is validated.
4. The final invoice is created. It carries the product line of 1 200.00 and a deduction line of −360.00 on 211100 with the same tax. Untaxed 840.00, tax 176.40, total 1 016.40.

| Account | Debit | Credit |
|---|---:|---:|
| 121000 Trade Receivables | 1 016.40 | |
| 400000 Product Sales | | 1 200.00 |
| 211100 Down Payments Received | 360.00 | |
| 251000 Tax Received | | 176.40 |
| 500000 Cost of Goods Sold | 480.00 | |
| 110300 Stock Valuation | | 480.00 |

Debits 1 856.40, credits 1 856.40. The two invoices together receive 435.60 + 1 016.40 = 1 452.00, exactly the order total, and 211100 returns to zero.

### GS-04: Wholesale pricelist and a global discount, in both discount modes

**Purpose.** Pricelist resolution by minimum quantity and category, and the two ways of applying a global discount.

1. Deltatech Solutions is switched to the pricelist Wholesale. An order of 12 Office Desk is created. The rule on the category "All / Furniture" applies because the quantity 12 reaches the minimum of 10: unit price `round_to(300.00 × 0.90, 0.01) = 270.00`. Untaxed 3 240.00.
2. A global discount of five percent is applied in the mode "percentage on every line". The line discount becomes 5.00 percent, the line subtotal `round_to(12 × 270.00 × 0.95, 0.01) = 3 078.00`, tax 646.38, total 3 724.38.
3. The same order with the discount applied in the mode "one discount line" keeps the product line at 3 240.00 and adds a line on the Sales Discounts account of −162.00 carrying Sales Tax 21%. Untaxed 3 078.00, tax 646.38, total 3 724.38. The two modes produce the same totals and different journal items: the first credits 400000 with 3 078.00, the second credits 400000 with 3 240.00 and debits 400200 with 162.00.
4. Reducing the quantity to 9 re-resolves the pricelist: the minimum quantity is no longer reached, the fallback rule applies and the unit price returns to 300.00.

### GS-05: Export order in a foreign currency with an exchange loss

**Purpose.** Foreign currency documents, zero-rated export tax mapping and the exchange difference created at reconciliation.

1. On 2026-01-15 an order is created for Pacific Retail: currency United States dollar, pricelist Export, fiscal position Export. The rate in force is 1.1000.
2. One line of 10 Office Desk. The Export pricelist converts the sales price: `round_to(300.00 × 1.1000, 0.01) = 330.00` dollars. Line subtotal 3 300.00 dollars. The fiscal position maps Sales Tax 21% to Zero-Rated Export, therefore the tax is 0.00 and the income account 400000 is mapped to 400300.
3. The order is confirmed and delivered on 2026-01-18.
4. The invoice is posted on 2026-01-20, still at the rate 1.1000. Company amount `3 300.00 ÷ 1.1000 = 3 000.00`.

| Account | Currency amount | Debit | Credit |
|---|---:|---:|---:|
| 121000 Trade Receivables | 3 300.00 dollars | 3 000.00 | |
| 400300 Export and Intra-Community Sales | 3 300.00 dollars | | 3 000.00 |
| 500000 Cost of Goods Sold | | 1 200.00 | |
| 110300 Stock Valuation | | | 1 200.00 |

5. On 2026-02-10 the customer pays 3 300.00 dollars into the dollar bank account. The rate in force is 1.2500, therefore the company amount is `3 300.00 ÷ 1.2500 = 2 640.00`. The payment entry debits 101200 Outstanding Receipts 2 640.00 (3 300.00 dollars) and credits 121000 Trade Receivables 2 640.00 (3 300.00 dollars).
6. The receivable items are reconciled. The currency amounts cancel exactly, the company amounts leave a difference of `3 000.00 − 2 640.00 = 360.00`. One exchange difference entry is posted in the journal EXCH, dated 2026-02-10:

| Account | Currency amount | Debit | Credit |
|---|---:|---:|---:|
| 630000 Foreign Exchange Loss | | 360.00 | |
| 121000 Trade Receivables | 0.00 dollars | | 360.00 |

The invoice's payment state becomes `paid` and its residual in both currencies is 0.00.

### GS-06: Partial delivery, backorder and two invoices

**Purpose.** Backorders, the invoicing policy "delivered quantities" with partial deliveries, and the proof that splitting a document does not change the total tax.

1. An order for Meridian Bakery with 10 Desk Lamp at 19.99. Untaxed 199.90, tax `round_to(199.90 × 0.21, 0.01) = round_to(41.979, 0.01) = 41.98`, total 241.88.
2. The delivery is validated with 6 units. The backorder dialog is answered "create a backorder": `MW/OUT/00002` is set to done with 6 units, and `MW/OUT/00003` is created with 4 units in state `assigned`, linked to the first.
3. An invoice is created: 6 units, untaxed 119.94, tax `round_to(25.1874, 0.01) = 25.19`, total 145.13. Cost of goods sold `6 × 8.00 = 48.00` (standard costing).
4. The backorder is validated and a second invoice is created: 4 units, untaxed 79.96, tax `round_to(16.7916, 0.01) = 16.79`, total 96.75. Cost of goods sold 32.00.
5. The two invoices sum to 241.88 and their taxes sum to `25.19 + 16.79 = 41.98`, exactly the tax of the single-invoice case. The order's invoicing status becomes `invoiced`.

### GS-07: Credit note for a return, and the refund payment

**Purpose.** Returns, partial credit notes, the reversal of the cost of goods sold and an outbound payment to a customer.

**Predecessor:** GS-01.

1. On 2026-03-10 the customer returns 1 Office Chair. A return transfer is created from the delivery `MW/OUT/00001` with one line of 1 unit, and validated. The chair re-enters stock at the value it left with, 45.00.
2. A credit note is created from the invoice, keeping only the Office Chair line with the quantity 1. Posted on 2026-03-10 with the number `RINV/2026/00001`.

| Account | Debit | Credit |
|---|---:|---:|
| 400000 Product Sales | 99.00 | |
| 251000 Tax Received | 20.79 | |
| 121000 Trade Receivables | | 119.79 |
| 110300 Stock Valuation | 45.00 | |
| 500000 Cost of Goods Sold | | 45.00 |

```formula
tax = round_to(99.00 × 0.21, 0.01) = round_to(20.79, 0.01) = 20.79
```

3. Because the original invoice was already paid, the credit note leaves a credit balance of 119.79 on the customer. An outbound payment of 119.79 is registered and reconciled with the credit note: debit 121000 Trade Receivables 119.79, credit 101300 Outstanding Payments 119.79.
4. The credit note's payment state becomes `paid`; the customer's balance returns to zero.

### GS-08: Early payment discount under the three tax reduction modes

**Purpose.** The three modes of an early payment discount, on the invoice and on the payment.

**Common invoice.** Harbour Group, 4 Installation Service at 250.00, untaxed 1 000.00, Sales Tax 21%, invoice date 2026-02-02, terms with a two percent discount within ten days.

| Mode | Tax base on the invoice | Tax | Invoice total | Due if paid by 2026-02-12 | Due afterwards |
|---|---:|---:|---:|---:|---:|
| On early payment | 1 000.00 | 210.00 | 1 210.00 | `1 210.00 × 0.98 = 1 185.80` | 1 210.00 |
| Never | 1 000.00 | 210.00 | 1 210.00 | `1 210.00 − 1 000.00 × 0.02 = 1 190.00` | 1 210.00 |
| Always (upon invoice) | 980.00 | 205.80 | 1 205.80 | `1 205.80 − 1 000.00 × 0.02 = 1 185.80` | 1 205.80 |

**Mode "On early payment", payment of 1 185.80 on 2026-02-09:**

| Account | Debit | Credit |
|---|---:|---:|
| 101200 Outstanding Receipts | 1 185.80 | |
| 640000 Early Payment Discount Granted, carrying Sales Tax 21% and its grids | 20.00 | |
| 251000 Tax Received | 4.20 | |
| 121000 Trade Receivables | | 1 210.00 |

`20.00 = 1 000.00 × 0.02` and `4.20 = 210.00 × 0.02`; together 24.20, exactly `1 210.00 − 1 185.80`. The reported tax of the period falls by 4.20.

**Mode "Never", payment of 1 190.00:** outstanding receipts 1 190.00, one discount line of 20.00 on 640000 with no tax at all, receivable credit 1 210.00. The reported tax is unchanged.

**Mode "Always (upon invoice)", payment of 1 185.80:** the invoice already carries the two anticipated discount lines of −20.00 (with tax) and +20.00 (without tax); the payment entry contains outstanding receipts 1 185.80, one discount line of 20.00 with no tax and the receivable credit of 1 205.80.

**Paid late in mode "On early payment", on 2026-02-20:** the payment is 1 210.00, no discount line is produced, and the residual becomes zero.

---

## 6. Golden scenarios, procure to pay

### GS-09: Request for quotation, order, receipt, bill on ordered quantities

**Purpose.** The base purchasing chain and the posting of a vendor bill under perpetual valuation with average costing.

1. On 2026-02-02 a request for quotation is created for Baltic Timber with 20 Table Top. The vendor price rule applies (60.00 from a minimum of 10), the payment terms are 30 Days and the purchase tax is Purchase Tax 21%. Untaxed 1 200.00, tax 252.00, total 1 452.00. State `draft`, document number `P00001`.
2. The request is sent; the state becomes `sent` and the discussion thread records the sending.
3. The order is confirmed on 2026-02-02. State `purchase`, order date 2026-02-02, receipt `MW/IN/00001` created with the scheduled date 2026-02-07 (vendor lead time five days).
4. The receipt is validated on 2026-02-07 with 20 units. Stock of Table Top becomes 30. The valuation layer records 20 units at the order price, value 1 200.00. **No journal entry** is posted, because the source location `Vendors` carries no valuation account.
5. The vendor bill is created from the order. With the bill control policy "on ordered quantities" it proposes 20 units. It is posted on 2026-02-10 with the due date 2026-03-12.

| Account | Debit | Credit |
|---|---:|---:|
| 110300 Stock Valuation | 1 200.00 | |
| 131000 Tax Paid | 252.00 | |
| 211000 Trade Payables | | 1 452.00 |

6. The receipt layer is re-valued to the billed amount, which is unchanged at 1 200.00. The average cost of Table Top stays 60.00: `(600.00 + 1 200.00) ÷ 30 = 60.00`.
7. A payment of 1 452.00 is registered on 2026-03-12 and reconciled: debit 211000 Trade Payables 1 452.00, credit 101300 Outstanding Payments 1 452.00.

### GS-10: Bill control on received quantities with a partial receipt

**Purpose.** The two bill control policies and the over-billing guard.

1. An order for Lumina Lighting with 100 Desk Lamp at 8.00, bill control "on received quantities". Untaxed 800.00, tax 168.00, total 968.00.
2. The receipt is validated with 60 units and a backorder of 40 is created. The order's received quantity becomes 60 and its receipt status `partially received`.
3. Creating the bill proposes 60 units: untaxed 480.00, tax 100.80, total 580.80.
4. Raising the billed quantity to 100 on the draft bill and posting it produces the over-billing warning of the purchasing domain, and the order's billing status becomes `over-billed` once posted.
5. Repeating the scenario with the policy "on ordered quantities" proposes 100 units from the start, for 800.00 untaxed.

### GS-11: Price difference on a standard-cost product

**Purpose.** The price difference pair produced when the billed price differs from the standard cost.

1. An order for Lumina Lighting with 100 Desk Lamp at 8.50 (the vendor raised the price). Untaxed 850.00, tax 178.50, total 1 028.50.
2. The receipt of 100 units is validated. The movement value is `100 × 8.00 = 800.00`, because the costing method is standard price.
3. The bill of 100 units at 8.50 is posted.

| Account | Debit | Credit |
|---|---:|---:|
| 110300 Stock Valuation | 850.00 | |
| 131000 Tax Paid | 178.50 | |
| 211000 Trade Payables | | 1 028.50 |
| 500100 Purchase Price Difference | 50.00 | |
| 110300 Stock Valuation | | 50.00 |

The net movement on 110300 is 800.00, exactly the physical value `100 × 8.00`. The product cost stays 8.00.

4. Repeating the scenario with the price difference account removed from the category produces no price difference pair: 110300 carries 850.00 against a physical value of 800.00, and the next closing entry credits 110300 by 50.00 and debits 510000 Inventory Variation by 50.00.

### GS-12: Landed cost split by value, then by quantity

**Purpose.** Landed cost allocation, its entry and its effect on the average cost.

**Predecessor:** GS-09, and a second receipt of 40 Table Leg at 7.50 (value 300.00) billed and posted the same way.

1. A landed cost record is created for the freight bill of Crescent Logistics: one line on the Freight Landed Cost product for 300.00, split by value, applied to the two receipts.
2. The allocation is computed on the values of the receipts, 1 200.00 and 300.00, total 1 500.00:

```formula
Table Top receipt : 300.00 × 1200.00 ÷ 1500.00 = 240.00
Table Leg receipt : 300.00 ×  300.00 ÷ 1500.00 =  60.00
```

3. Validating the landed cost posts, in the journal STCK:

| Account | Debit | Credit |
|---|---:|---:|
| 110300 Stock Valuation, Table Top | 240.00 | |
| 110300 Stock Valuation, Table Leg | 60.00 | |
| 600000 Operating Expenses | | 300.00 |

4. The costs become: Table Top `(600.00 + 1 200.00 + 240.00) ÷ 30 = 68.00`; Table Leg `(300.00 + 300.00 + 60.00) ÷ 80 = 8.25`.
5. Repeating the scenario with the split method "by quantity" allocates on 20 and 40 units: `300.00 × 20 ÷ 60 = 100.00` and `300.00 × 40 ÷ 60 = 200.00`, giving costs of `(600 + 1200 + 100) ÷ 30 = 63.33` and `(300 + 300 + 200) ÷ 80 = 10.00`.
6. A split method whose total base is zero is refused with the message of the landed cost file, and nothing is posted.

### GS-13: Purchase in a foreign currency with a rate change at payment

**Purpose.** Foreign currency payables and the exchange loss on settlement.

1. On 2026-01-15 an order is created for Pacific Supplies in United States dollars: 10 Office Chair at 55.00 dollars, total 550.00 dollars, fiscal position Export, therefore no purchase tax. The rate in force is 1.1000, company amount 500.00.
2. The receipt is validated on 2026-01-25.
3. The bill is posted on 2026-02-05, when the rate is 1.2500: company amount `550.00 ÷ 1.2500 = 440.00`.

| Account | Currency amount | Debit | Credit |
|---|---:|---:|---:|
| 110300 Stock Valuation | 550.00 dollars | 440.00 | |
| 211000 Trade Payables | 550.00 dollars | | 440.00 |

The receipt layer is re-valued to 440.00 and the average cost of Office Chair becomes `(1 800.00 + 440.00) ÷ 50 = 44.80`.

4. On 2026-03-05 the bill is paid, at the rate 1.0000: company amount 550.00. The payment entry debits 211000 Trade Payables 550.00 (550.00 dollars) and credits 101300 Outstanding Payments 550.00 (550.00 dollars).
5. Reconciliation leaves a company-amount difference of 110.00 and posts the exchange entry: debit 630000 Foreign Exchange Loss 110.00, credit 211000 Trade Payables 110.00 with a currency amount of 0.00.

### GS-14: Intra-community purchase with the reverse charge

**Purpose.** A tax that produces two opposite lines, and the grids it stamps.

1. A bill is created for Nordic Components, whose fiscal position Intra-Community maps Purchase Tax 21% to Intra-Community Purchase 21%. One line of 20 Office Chair at 45.00, untaxed 900.00.
2. The tax expands into its two children, +21 percent and −21 percent, producing two tax lines of 189.00 each, one on 131000 and one on 251000.

| Account | Debit | Credit |
|---|---:|---:|
| 110300 Stock Valuation | 900.00 | |
| 131000 Tax Paid | 189.00 | |
| 251000 Tax Received | | 189.00 |
| 211000 Trade Payables | | 900.00 |

3. The base line carries the grid `+86`, the deductible tax line the grid `+59` and the due tax line the grid `+56`. The net effect on the result of the period is zero and the payable is 900.00.
4. The tax return of the period shows 900.00 in the intra-community base line, 189.00 in the due line and 189.00 in the deductible line.

---

## 7. Golden scenarios, replenishment and routes

### GS-15: A reordering rule triggers a purchase

**Purpose.** The scheduler, the ordering quantity arithmetic and the backward date computation.

**Predecessor:** GS-12, so that 80 Table Leg are on hand.

1. A manufacturing order for 15 Dining Tables is confirmed on 2026-02-16, reserving 60 Table Leg. The forecast quantity of Table Leg becomes `80 − 60 = 20`.
2. The scheduler runs on 2026-02-16. The reordering rule of Table Leg (minimum 40, maximum 120, multiple of 20) is below its minimum:

```formula
missing        = maximum − forecast = 120 − 20 = 100
ordered        = round up 100 to a multiple of 20 = 100
```

3. The Buy rule creates a request for quotation for Baltic Timber with 100 Table Leg at 7.50, untaxed 750.00, with the scheduled receipt date 2026-02-21 and the order deadline computed backwards from it by the vendor lead time of five days, that is 2026-02-16.
4. Running the scheduler a second time on the same day creates nothing, because the forecast now includes the incoming 100 units.
5. Snoozing the rule until 2026-03-01 and dropping the forecast further makes the scheduler skip it until that date.

### GS-16: Make to order with drop shipping

**Purpose.** A supply created for one demand and a movement that never touches the warehouse.

1. An order for Meridian Bakery with 20 Desk Lamp is confirmed on 2026-02-20. The Dropship route applies.
2. One purchase order is created for Lumina Lighting with 20 Desk Lamp at 8.00, whose delivery address is the customer's address, and one dropship transfer from `Vendors` to `Customers` linked to both documents.
3. Validating the dropship transfer records an incoming and an outgoing movement of 20 units. The quantity on hand of Desk Lamp in the warehouse is unchanged at 100.
4. The valuation writes an incoming layer of `20 × 8.00 = 160.00` and an outgoing layer of −160.00; the net effect on 110300 is zero once the vendor bill is posted, and the cost of goods sold of 160.00 is recognized when the customer invoice is posted.
5. Cancelling the sales order before the dropship is validated cancels the purchase order line and the transfer.

### GS-17: Two-step delivery from the second warehouse

**Purpose.** Chained transfers, propagation of dates and of cancellation.

1. An order for Deltatech Solutions with 5 Office Chair, delivered from South Depot, is confirmed. Two transfers are created: `SD/PICK/00001` from `SD/Stock` to `SD/Output`, and `SD/OUT/00001` from `SD/Output` to `Customers`, chained, the second waiting for the first.
2. Rescheduling the order's delivery date by two days reschedules both transfers and writes the rescheduling message on them.
3. Validating the pick makes the ship `assigned`. The goods sit in `SD/Output`, still owned by the company, and the valuation is unchanged because both locations are internal.
4. Cancelling the ship leaves the goods in `SD/Output` and, when the rule declares propagation, cancels the pick as well; otherwise the pick stays done.

---

## 8. Golden scenarios, inventory operations

### GS-18: Lots with expiration, first expired first out removal, and the effect of lot valuation

**Purpose.** Lot tracking, the removal strategy and the difference between the removal order and the valuation order.

1. On 2026-02-05 a receipt of 30 kilograms of Coffee Beans is validated for the lot `L-2026-B` with the expiration date 2026-04-30, at a purchase price of 13.00 per kilogram. Stock: lot `L-2026-A` 50 kilograms (opening, 12.00, expires 2026-06-30) and lot `L-2026-B` 30 kilograms (13.00, expires 2026-04-30).
2. An order for Meridian Bakery with 60 kilograms is confirmed and reserved. The removal strategy of the category "All / Beverages" is first expired first out, therefore the reservation takes 30 kilograms of `L-2026-B` first and 30 kilograms of `L-2026-A`.
3. The delivery is validated. **Valuation without lot valuation** consumes the layers in first in first out order regardless of the lots:

```formula
from the opening layer : 50 kg × 12.00 = 600.00
from the receipt layer : 10 kg × 13.00 = 130.00
value of the delivery                  = 730.00
remaining: 20 kg of the receipt layer worth 260.00, cost 13.00 per kilogram
```

4. **Valuation with lot valuation enabled on the category** follows the lots that were physically removed:

```formula
lot L-2026-B : 30 kg × 13.00 = 390.00
lot L-2026-A : 30 kg × 12.00 = 360.00
value of the delivery        = 750.00
remaining: 20 kg of lot L-2026-A worth 240.00
```

5. The invoice of 60 kilograms at 25.00 with Sales Tax 6%: untaxed 1 500.00, tax 90.00, total 1 590.00. The cost of goods sold line is 730.00 in the first variant and 750.00 in the second.
6. On 2026-05-01 the remaining quantity of `L-2026-B`, if any, is past its expiration date and is excluded from reservations by the expiry rule, with the warning of the lots file.

### GS-19: Inventory adjustment with a loss

**Purpose.** Counting, the adjustment entry and the conflict guard.

**Predecessor:** GS-01, so that 36 Office Chair are on hand.

1. A count request is created for `MW/Stock/Shelf A`. The counted quantity of Office Chair is entered as 34.
2. Applying the adjustment creates a move of 2 units from `MW/Stock/Shelf A` to `Inventory adjustment`, valued at the average cost `2 × 45.00 = 90.00`.

| Account | Debit | Credit |
|---|---:|---:|
| 510000 Inventory Variation | 90.00 | |
| 110300 Stock Valuation | | 90.00 |

3. A second user who opened the same quantity record before the adjustment and applies a count of 35 receives the conflict dialog and must re-read the record; nothing is written until the conflict is resolved.
4. Counting 38 instead produces a gain of 2 units valued at the current average cost, debiting 110300 by 90.00 and crediting 510000.

### GS-20: Internal transfer between warehouses

**Purpose.** That an internal movement changes location quantities and nothing else.

1. An internal transfer of 10 Office Chair from `MW/Stock/Shelf A` to `SD/Stock` is created and validated.
2. Quantities: Main Warehouse 26, South Depot 10. The company total is unchanged.
3. No journal entry is posted and no valuation layer is created, because both locations are internal and neither carries a valuation account.
4. The average cost of Office Chair is unchanged.

### GS-21: Scrap

**Purpose.** The scrap entry and the guard on the available quantity.

1. One Desk Lamp is scrapped from `MW/Stock/Shelf A` with the reason "Damaged in handling".

| Account | Debit | Credit |
|---|---:|---:|
| 510000 Inventory Variation | 8.00 | |
| 110300 Stock Valuation | | 8.00 |

2. Scrapping 200 units, more than the 99 remaining, is refused with the insufficient quantity message of the inventory domain and nothing is written.
3. The scrapped unit is visible in the traceability report of the product with the document that scrapped it.

---

## 9. Golden scenarios, make to stock

### GS-22: Manufacture five dining tables

**Purpose.** Explosion, component consumption, work center cost, the production account and the finished cost.

**Predecessor:** GS-12, so that Table Top costs 68.00 and Table Leg costs 8.25.

1. A manufacturing order for 5 Dining Table is created on 2026-02-16. The explosion of the bill of materials produces the component moves: 5 Table Top and 20 Table Leg. One work order "Assembly" is created on the work center Assembly Line with an expected duration of `5 × 30 = 150` minutes.
2. The order is confirmed and the components are reserved. State `confirmed`, then `progress` when the work order starts.
3. The work order is started at 09:00 and finished at 11:30 on 2026-02-17: 150 minutes recorded.
4. Five serial numbers are assigned to the finished product and the order is marked done.
5. The costing routine computes:

```formula
component value = 5 × 68.00 + 20 × 8.25 = 340.00 + 165.00 = 505.00
work center cost = 150 ÷ 60 × 45.00 = 112.50
total cost       = 505.00 + 112.50 = 617.50
finished unit cost = 617.50 ÷ 5 = 123.50
```

6. Three entries are posted in the journal STCK:

| Entry | Account | Debit | Credit |
|---|---|---:|---:|
| Components, Table Top | 110400 Production | 340.00 | |
| | 110300 Stock Valuation | | 340.00 |
| Components, Table Leg | 110400 Production | 165.00 | |
| | 110300 Stock Valuation | | 165.00 |
| Labour | 110400 Production | 112.50 | |
| | 610100 Work Center Expenses | | 112.50 |
| Finished product | 110300 Stock Valuation | 617.50 | |
| | 110400 Production | | 617.50 |

7. The balance of 110400 Production returns to zero. The average cost of Dining Table becomes 123.50 and the stock is five units for 617.50.
8. Posting the labour entry a second time is refused because every productivity record already carries its journal item.

### GS-23: Manufacturing backorder

**Purpose.** Producing less than planned and splitting the remainder.

1. A manufacturing order for 10 Dining Table is confirmed with the same costs as GS-22.
2. Six units are produced. The backorder dialog is answered "create a backorder".
3. The first order closes with 6 units; the components consumed are 6 Table Top and 24 Table Leg; the work order records 180 minutes.

```formula
component value = 6 × 68.00 + 24 × 8.25 = 408.00 + 198.00 = 606.00
work center cost = 180 ÷ 60 × 45.00 = 135.00
total cost = 741.00 ; finished unit cost = 741.00 ÷ 6 = 123.50
```

4. The backorder carries 4 units with its own component moves of 4 Table Top and 16 Table Leg, in state `confirmed`, linked to the first order.
5. The sum of the two orders' component quantities equals the original plan exactly: 10 Table Top and 40 Table Leg.

### GS-24: Unbuild

**Purpose.** Reversing a production and what stays on the production account.

**Predecessor:** GS-22.

1. An unbuild order naming the manufacturing order of GS-22 is created for 1 Dining Table and validated.
2. The finished unit is consumed at the value produced by that order, 123.50, and the components return at the values they had when that order consumed them: 1 Table Top at 68.00 and 4 Table Leg at 33.00.

| Entry | Account | Debit | Credit |
|---|---|---:|---:|
| Finished product consumed | 110400 Production | 123.50 | |
| | 110300 Stock Valuation | | 123.50 |
| Components returned, Table Top | 110300 Stock Valuation | 68.00 | |
| | 110400 Production | | 68.00 |
| Components returned, Table Leg | 110300 Stock Valuation | 33.00 | |
| | 110400 Production | | 33.00 |

3. The production account keeps a debit balance of `123.50 − 101.00 = 22.50`, which is the labour that was capitalized into the unbuilt unit and is now released. The specification does not clear it automatically; an accountant posts the charge.
4. Unbuilding two units when only one was produced is refused with the insufficient quantity message.

### GS-25: Selling a kit

**Purpose.** A kit bill of materials exploded at delivery and never manufactured.

1. An order for Deltatech Solutions with 2 Desk Set at 399.00. Untaxed 798.00, tax 167.58, total 965.58.
2. Confirming creates one delivery with three move lines: 2 Office Desk, 2 Office Chair, 2 Desk Lamp. No manufacturing order is created.
3. The delivery is validated. Valuation: `2 × 120.00 + 2 × 45.00 + 2 × 8.00 = 240.00 + 90.00 + 16.00 = 346.00`.
4. The invoice is posted: one line for the kit at 798.00 on 400000, the tax of 167.58, the receivable of 965.58 and the cost of goods sold of 346.00 against 110300.
5. Setting a reordering rule on a kit is refused with the message of the manufacturing domain.

### GS-26: Subcontracted production

**Purpose.** Components sent to a subcontractor, a finished product received, and the production account netting to zero.

1. A subcontracting bill of materials for Dining Table names Nordic Components as the subcontractor.
2. A purchase order for 3 Dining Table at 90.00 each is confirmed, total 270.00. A resupply transfer sends 3 Table Top and 12 Table Leg to the subcontractor's location.
3. The resupply is validated: component value `3 × 68.00 + 12 × 8.25 = 204.00 + 99.00 = 303.00`.
4. The receipt of 3 finished tables is validated. The subcontracting production consumes the components and produces the finished goods at:

```formula
finished total cost = component value + subcontracting price = 303.00 + 270.00 = 573.00
finished unit cost  = 573.00 ÷ 3 = 191.00
```

5. After the vendor bill of 270.00 is posted, the production account of the subcontracting location nets to zero and 110300 carries 573.00 for the three tables. The exact account pairs are those of [`../domains/manufacturing/workflows.md`](../domains/manufacturing/workflows.md); the invariants to check are the zero production balance and the valuation of 573.00.
6. The traceability report of a finished serial number lists the components consumed at the subcontractor.

---

## 10. Golden scenarios, a point of sale day

### GS-27: Open, sell four orders, close with a cash difference

**Purpose.** The complete counter day and its single closing entry.

1. At 08:00 on 2026-03-02 the cashier opens a session on the counter Front Shop and confirms the opening cash balance of 200.00.
2. Four orders are taken. The counter's fiscal position makes every price tax-included.

| Order | Content | Total | Base | Tax | Payment |
|---|---|---:|---:|---:|---|
| 1 | 2 kilograms Coffee Beans at 25.00 | 50.00 | 47.17 | 2.83 at 6 percent | cash 50.00 |
| 2 | 1 Office Chair at 99.00 | 99.00 | 81.82 | 17.18 at 21 percent | card 99.00 |
| 3 | 3 Desk Lamp at 19.99 | 59.97 | 49.56 | 10.41 at 21 percent | cash, rounded to 59.95 |
| 4 | 1 Office Desk at 300.00 | 300.00 | 247.93 | 52.07 at 21 percent | customer account of Deltatech Solutions |

```formula
order 1 tax = round_to(50.00 × (1 ÷ 1.06) × 0.06, 0.01) = round_to(2.830188, 0.01) = 2.83
order 2 tax = round_to(99.00 × (1 ÷ 1.21) × 0.21, 0.01) = round_to(17.181818, 0.01) = 17.18
order 3 tax = round_to(59.97 × (1 ÷ 1.21) × 0.21, 0.01) = round_to(10.408016, 0.01) = 10.41
order 4 tax = round_to(300.00 × (1 ÷ 1.21) × 0.21, 0.01) = round_to(52.066116, 0.01) = 52.07
cash rounding of order 3 = round_to(59.97, 0.05) − 59.97 = 59.95 − 59.97 = −0.02
```

3. The cashier counts 309.45 in the drawer. The expected cash is `200.00 + 50.00 + 59.95 = 309.95`, therefore the difference is −0.50.
4. Closing the session posts one entry in the journal PSAL, dated 2026-03-02:

| Account | Debit | Credit |
|---|---:|---:|
| 101000 Cash on Hand | 109.95 | |
| 101200 Outstanding Receipts (card) | 99.00 | |
| 271000 Point of Sale Receivable | 300.00 | |
| 620200 Cash Rounding | 0.02 | |
| 400000 Product Sales | | 426.48 |
| 251000 Tax Received | | 82.49 |
| 500000 Cost of Goods Sold | 213.00 | |
| 110300 Stock Valuation | | 213.00 |

```formula
sales base   = 47.17 + 81.82 + 49.56 + 247.93 = 426.48
tax          =  2.83 + 17.18 + 10.41 +  52.07 =  82.49
cost of goods sold = 2 × 12.00 + 45.00 + 3 × 8.00 + 120.00 = 24.00 + 45.00 + 24.00 + 120.00 = 213.00
debits  = 109.95 + 99.00 + 300.00 + 0.02 + 213.00 = 721.97
credits = 426.48 + 82.49 + 213.00 = 721.97
```

5. A second entry records the counting difference: debit 620000 Cash Difference Loss 0.50, credit 101000 Cash on Hand 0.50.
6. One delivery is created for the goods sold and validated, moving 2 kilograms of Coffee Beans, 1 Office Chair, 3 Desk Lamp and 1 Office Desk out of `MW/Stock`.
7. The session state becomes `closed and posted`. Re-closing it is refused.

### GS-28: Refund and invoice at the counter

**Purpose.** A refund order and an invoiced counter order.

**Predecessor:** GS-27, with the session reopened for the following day.

1. The customer returns the Office Chair of order 2. The cashier opens order 2 and refunds it: a new order is created with a quantity of −1, a total of −99.00, and a card payment of −99.00.
2. The refund order references the refunded order and the refunded order references the refund.
3. A fifth order of 1 Office Desk is taken for Deltatech Solutions and invoiced immediately. The invoice is posted in the journal INV with the base of 247.93, the tax of 52.07 and the receivable of 300.00; the cost of goods sold of 120.00 is recognized on the invoice.
4. At closing, the invoiced order contributes nothing to the sales and cost accumulators of the session entry; only its receivable reaches the entry, as a line on 271000 labeled "From invoice payments".

### GS-29: Restaurant table service with a split bill

**Purpose.** Tables, courses and the split of one order into two bills.

1. The waiter assigns table 5 of the floor Main Room to a new order and adds 4 Coffee Beans portions and 2 Paper Ream (used here as a sellable item), for a tax-included total of 112.00.
2. Two courses are fired to the preparation display; each appears once and is marked done once.
3. The order is transferred to table 6: the order keeps its lines, its payments and its reference, and table 5 becomes free.
4. The bill is split into two orders of 56.00 each. The sum of the two totals equals the original total exactly, the taxes are recomputed per split order, and the original order is closed as split.
5. Each split order is paid separately, one in cash and one by card.

### GS-30: Promotion and gift card at the counter

**Purpose.** The identical behavior of a promotion at the counter and on a sales order, and the gift card liability.

1. An order of 3 Office Desk at 300.00 tax-included totals 900.00, with a base of 743.80 and a tax of 156.20.
2. The promotion "Spring Promotion" applies because the untaxed amount 743.80 reaches 500.00. The reward line is a discount of ten percent of the order, that is 90.00 including tax.

```formula
discount including tax = round_to(900.00 × 0.10, 0.01) = 90.00
discount base          = round_to(90.00 × (1 ÷ 1.21), 0.01) = 74.38
discount tax           = 90.00 − 74.38 = 15.62
order total            = 900.00 − 90.00 = 810.00
order base             = 743.80 − 74.38 = 669.42
order tax              = 156.20 − 15.62 = 140.58
```

3. A gift card of 50.00 is sold on a separate order. The session entry credits 211200 Gift Card Liability with 50.00 and no tax is computed.
4. On a later order the gift card is redeemed for 50.00: the session entry debits 211200 Gift Card Liability with 50.00 and the remaining balance of the card is zero. A second redemption of the same card is refused with the message of the loyalty domain.
5. The same promotion applied to a sales order of 3 Office Desk at the tax-excluded price of 300.00 produces a discount line of −90.00 excluding tax and the same relative result, proving that the program is shared and not duplicated.

---

## 11. Golden scenarios, hire to retire

### GS-31: From applicant to employee

**Purpose.** The recruitment pipeline and the creation of an employee.

1. A job position "Warehouse Operator" is published with one expected recruitment.
2. An application arrives through the public form for "Nadia Hassan" with a curriculum attachment. An Applicant record is created in the first stage, with the source recorded and the attachment stored.
3. The applicant is moved to "First Interview", then "Contract Proposal". Each move writes the stage change in the discussion thread and updates the stage duration tracking.
4. An offer is recorded and the applicant is hired. An Employee record is created with the name, the work contact, the department, the job position and the working schedule of the position; the applicant record links to it; the job position's remaining recruitment count becomes zero and its state becomes `recruitment closed`.
5. A second applicant is refused with the reason "Not enough experience"; the refusal message is sent and the applicant is archived, not deleted.

### GS-32: Time off accrual, request and approval

**Purpose.** Accrual arithmetic, duration computation and the approval path.

1. Clara Mensah holds an allocation of 20 days of "Paid Time Off" for 2026 and an accrual plan "Two Days a Month" granting two days on the first day of every month, capped at 24 days.
2. The accrual job runs on 2026-02-01 and 2026-03-01. Her accrued allocation is `2 × 2 = 4` days; her total entitlement is `20 + 4 = 24` days.
3. Running the job twice on 2026-03-01 credits nothing the second time.
4. She requests time off from 2026-03-10 to 2026-03-12, three working days. The duration is computed on her working schedule: `3 days = 24 hours`.
5. The request needs one approval. Her manager approves it. The request state becomes `approved`, the remaining entitlement becomes `24 − 3 = 21` days, a calendar event is created for the period, and the work entries of those three days become time-off entries.
6. Requesting a fourth day on 2026-03-13 while 21 days remain is accepted; requesting 25 days is refused with the message of the time off domain.
7. Refusing the approved request after the fact restores the 3 days and notifies the employee.
8. Requesting time off on 2026-05-01, a public holiday, is refused because the day is not a working day.

### GS-33: Attendance and overtime

**Purpose.** Attendance pairing and the overtime computation.

1. On Monday 2026-03-02 Anna Lindqvist checks in at 09:00 and out at 12:00, then in at 13:00 and out at 18:30.
2. Two Attendance records are created with durations of 3.00 and 5.50 hours; the worked time of the day is 8.50 hours.
3. Her schedule expects 8.00 hours that day, therefore an overtime line of `8.50 − 8.00 = 0.50` hour is created in state "to approve".
4. Checking in a third time without checking out is refused with the message of the attendances domain.
5. An attendance left open beyond the configured maximum is closed by the automatic check-out job, which records that the closure was automatic.
6. On a day of approved time off, no absence is created by the absence detection job.

### GS-34: Departure

**Purpose.** Closing an employment without losing history.

1. Bruno Ferraro leaves on 2026-06-30 with the reason "Resigned".
2. The departure wizard archives the employee on 2026-07-01, closes the current employee version with the end date 2026-06-30, and cancels his future approved time off.
3. His attendances, work entries, expenses and timesheets remain readable and remain linked to him.
4. His user account, when he has one, is deactivated but not deleted; his documents keep their author.

---

## 12. Golden scenarios, expense to reimbursement

### GS-35: An expense paid by the employee, reimbursed

**Purpose.** The employee payable and its settlement.

1. Anna Lindqvist records an expense "Train ticket to client" of 121.00 including Sales Tax 21%, paid by herself, on 2026-03-04, with the receipt attached.

```formula
base = round_to(121.00 × (1 ÷ 1.21), 0.01) = 100.00
tax  = 121.00 − 100.00 = 21.00
```

2. She submits it; her manager approves it; the accountant posts it.

| Account | Partner | Debit | Credit |
|---|---|---:|---:|
| 600100 Travel Expenses | | 100.00 | |
| 131000 Tax Paid | | 21.00 | |
| 261000 Employee Payables | Anna Lindqvist | | 121.00 |

3. A payment of 121.00 is registered to the employee and reconciled with the payable: debit 261000 Employee Payables 121.00, credit 101300 Outstanding Payments 121.00. The expense state becomes `paid`.
4. Submitting the same receipt twice raises the duplicate approval warning and lists the earlier expense.
5. Refusing the expense before posting returns it to the employee with the reason recorded and posts nothing.

### GS-36: A company-paid expense and a mileage expense

**Purpose.** The second payment mode and an expense computed per unit.

1. Anna records a meal of 60.50 including Sales Tax 6 percent, paid with the company card.

```formula
base = round_to(60.50 × (1 ÷ 1.06), 0.01) = 57.08
tax  = 60.50 − 57.08 = 3.42
```

| Account | Debit | Credit |
|---|---:|---:|
| 600200 Meal Expenses | 57.08 | |
| 131000 Tax Paid | 3.42 | |
| 101300 Outstanding Payments | | 60.50 |

2. No employee payable is created; the outstanding line is cleared when the card statement is reconciled.
3. She records a mileage expense of 120 kilometres at 0.35 per kilometre: `120 × 0.35 = 42.00`, no tax, posted as debit 600100 Travel Expenses 42.00 and credit 261000 Employee Payables 42.00.
4. Splitting the meal expense into two of 30.25 produces two expenses whose bases are 28.54 and 28.54 and whose taxes are 1.71 and 1.71, and whose totals sum to 60.50 exactly.

---

## 13. Golden scenarios, lead to opportunity to order

### GS-37: Lead, opportunity, quotation, won

**Purpose.** The demand pipeline and its hand-over to sales.

1. A lead "Fit-out for a new office" arrives through the contact form with the electronic mail address of Deltatech Solutions. It is created in the first stage, unassigned, with the source recorded.
2. The assignment job assigns it to the team "Sales" and to Anna Lindqvist, respecting her maximum of leads.
3. The lead is converted into an opportunity, linked to the existing contact Deltatech Solutions because the address matches, with an expected revenue of 5 000.00 and the discussion thread preserved.
4. The automated probability is computed from the scoring frequencies; setting the probability manually to 60 stops the automatic recomputation for that record.
5. A quotation is created from the opportunity. It carries the customer, the salesperson, the team and the campaign fields, and the opportunity shows the quotation in its linked documents.
6. The quotation is confirmed. Marking the opportunity won sets its probability to 100 and moves it to the won stage. The order remains linked.

### GS-38: Lost, restored and merged

**Purpose.** Losing, restoring and merging opportunities.

1. A second opportunity for the same customer is marked lost with the reason "Too expensive". Its state records the lost reason, its probability becomes 0 and it leaves the open pipeline.
2. Restoring it returns it to its previous stage with its previous probability and clears the lost reason.
3. The two opportunities are merged. The survivor is the oldest one; the descriptions are concatenated; the activities, messages and attachments of the other are moved; the other is deleted.
4. Merging opportunities of two different customers asks for confirmation and keeps the customer of the survivor.

---

## 14. Golden scenarios, event registration to invoice

### GS-39: Selling event seats on a sales order

**Purpose.** The link between an order line and registrations.

1. An order for Deltatech Solutions with 4 Event Seat for the event "Spring Product Days", ticket "Standard Seat" at 150.00. Untaxed 600.00, tax 126.00, total 726.00.
2. Confirming the order creates four Event Registration records in state `open`, one per seat, each linked to the order line, and the ticket's remaining seats fall from 100 to 96.
3. The attendee details of each registration are completed through the attendee dialog.
4. The invoice is posted: receivable 726.00, income 600.00 on 400100, tax 126.00.
5. Four badges are printed, each with the attendee name and the scanning code.
6. Scanning a badge at the entrance marks that registration `attended` and records the moment; scanning it again reports that it is already attended and changes nothing.
7. Cancelling the sales order cancels the four registrations and returns the four seats to the ticket.

### GS-40: Free registration to a published event

**Purpose.** Registration without a sale, capacity and communication.

1. The event "Open House" is published with a free ticket and no seat limit.
2. A visitor registers through the public page with a name and an electronic mail address. One Event Registration is created in state `confirmed`, and the registration confirmation message is sent once.
3. The event communication configured "three days before the event" is sent by the event mail scheduler on 2026-05-17 to every confirmed registration, once.
4. Registering the same electronic mail address twice creates a second registration and raises the duplicate warning of the events domain, which does not block it.
5. Setting the ticket's seat limit to 1 after the second registration does not cancel it, and a third registration is refused with the sold-out message.

---

## 15. Golden scenarios, storefront checkout and portal

### GS-41: Cart, delivery method, online payment

**Purpose.** The storefront checkout from cart to confirmed order.

1. A visitor adds 3 Desk Lamp to the cart. A draft sales order is created for the public user with one line of 3 units at 19.99: untaxed 59.97, tax 12.59, total 72.56.

```formula
tax = round_to(59.97 × 0.21, 0.01) = round_to(12.5937, 0.01) = 12.59
```

2. Adding the same product again increases the line to 4 units rather than adding a second line.
3. The quantity is set back to 3. The visitor signs in as Deltatech Solutions; the cart takes the customer's pricelist, fiscal position and addresses, and the prices are recomputed.
4. The delivery method "Standard Delivery" is chosen. A delivery line of 15.00 carrying Sales Tax 21% is added: untaxed 74.97, tax 15.74, total 90.71.

```formula
tax = round_to(74.97 × 0.21, 0.01) = round_to(15.7437, 0.01) = 15.74
```

5. The visitor pays with a card through the payment provider. A transaction is created, the provider confirms it, the order is confirmed, a payment of 90.71 is created and the cart is emptied.
6. The confirmed order creates the delivery with three lamps and one service line that creates no move.
7. A cart abandoned before payment is detected by the abandoned-cart job after the configured delay and one recovery message is sent, once.

### GS-42: Coupon on the cart and the out-of-stock guard

**Purpose.** Promotions on the storefront and the availability setting.

1. A cart of 3 Desk Lamp, untaxed 59.97.
2. The coupon code `WELCOME10` is entered. A reward line of −6.00 carrying Sales Tax 21% is added.

```formula
reward = round_to(−59.97 × 0.10, 0.01) = round_to(−5.997, 0.01) = −6.00
untaxed after reward = 53.97
tax = round_to(53.97 × 0.21, 0.01) = round_to(11.3337, 0.01) = 11.33
total = 65.30
```

3. Entering the same code again is refused with the message of the loyalty domain; the reward line is not duplicated.
4. With the setting "allow orders beyond the available quantity" turned off, adding 200 Desk Lamp when 100 are available is refused with the declared message and the line keeps the available quantity.
5. With the setting turned on, the line accepts 200 and the storefront shows the delivery delay warning.
6. Subscribing to the back-in-stock notification for an out-of-stock product sends exactly one message when the quantity becomes positive.

### GS-43: Paying an invoice in the portal

**Purpose.** Portal access, token sharing and online payment of an existing document.

**Predecessor:** GS-01, with the payment step not performed.

1. The customer's contact is granted portal access. An invitation is sent with a link; the contact sets a password and signs in.
2. The portal lists exactly the documents of Deltatech Solutions: one order, one delivery and one invoice. A document of Meridian Bakery is not listed and is refused when its address is guessed.
3. The invoice page shows the total 1 205.16 and the residual 1 205.16, with the printed document downloadable.
4. The customer pays by card. A transaction for exactly 1 205.16 is created, confirmed, and a payment is created and reconciled with the invoice. The residual becomes 0.00 and the payment state becomes `paid`.
5. A link shared by access token opens the same document without signing in, and revoking the token makes the link refuse access.

---

## 16. Golden scenarios, projects and timesheets

### GS-44: Billable timesheets

**Purpose.** Time logged against a task becoming a delivered quantity and then an invoice.

1. An order for Deltatech Solutions with 40 Consulting Hour at 120.00 and the invoicing policy "based on timesheets" is confirmed. Untaxed 4 800.00, tax 1 008.00, total 5 808.00. A project "Deltatech Rollout" and a task are created from the service line, both linked to the order line and to a new analytic account.
2. Clara Mensah logs 12.5 hours on the task across three days. Each log creates an analytic line on the project's analytic account with a cost of `hours × 60.00`, that is −750.00 in total, and increases the delivered quantity of the order line to 12.5.
3. An invoice is created for the delivered quantity: `12.5 × 120.00 = 1 500.00` untaxed, tax 315.00, total 1 815.00. The invoiced quantity becomes 12.5 and the timesheet lines are marked billed.
4. Posting the invoice creates the revenue analytic line of 1 500.00 on the same analytic account. The project's margin is `1 500.00 − 750.00 = 750.00`.
5. Editing a billed timesheet line does not change the posted invoice; the difference appears as a quantity to invoice or to credit on the order line.
6. Deleting Clara while she has timesheets is refused, and the transfer dialog reassigns them to another employee.

### GS-45: Milestone invoicing

**Purpose.** Invoicing on events rather than on quantities.

1. An order for Harbour Group with three milestone lines on a service product with the invoicing policy "based on milestones": 30 percent, 40 percent and 30 percent of 10 000.00, that is 3 000.00, 4 000.00 and 3 000.00 excluding tax.
2. Reaching the first milestone sets its delivered quantity to 1 and makes 3 000.00 invoiceable.
3. The invoice is posted: untaxed 3 000.00, tax 630.00, total 3 630.00.
4. The remaining invoiceable amount is 7 000.00 and the order's invoicing status stays `to invoice`.
5. Reaching a milestone twice does not make it invoiceable twice.

---

## 17. Golden scenarios, period close

### GS-46: Month-end close of February

**Purpose.** The periodic tasks and their interaction.

**Predecessor:** GS-01 to GS-14 executed in order.

1. **Unrealized currency revaluation at 2026-02-28.** The receivable of Pacific Retail is 3 300.00 dollars booked at 3 000.00; the rate in force on 2026-02-28 is 1.2500, therefore the current value is 2 640.00 and the unrealized loss is 360.00. One entry is posted on 2026-02-28 in the journal EXCH, debit 630000 Foreign Exchange Loss 360.00 and credit 121000 Trade Receivables 360.00 with a currency amount of 0.00, and a reversing entry is posted on 2026-03-01.
2. **Tax return for February.** The tax report aggregates the grids of the period: the sales bases on `+03` and `+01`, the sales tax on `+54`, the purchase bases on `+81` and `+86`, the deductible tax on `+59` and the reverse-charge tax due on `+56`. The balance to pay equals `tax due − deductible tax`, and closing the return posts the entry that moves the balances to the tax payable account and locks the period for tax.
3. **Inventory valuation closing.** Every category of the fixture is valued automatically, therefore the closing job posts nothing, and the check `balance of 110300 = sum of the layer values` holds.
4. **Consistency tests.** The six shipped accounting tests report zero anomalies.
5. **Reporting.** The trial balance, the balance sheet, the income statement, the aged receivable and the aged payable are produced and are internally consistent: the balance sheet's result equals the income statement's result, the aged receivable total equals the balance of 121000, the aged payable total equals the balance of 211000.

### GS-47: Lock date, exception and inalterability

**Purpose.** The guards that make the books auditable.

1. The accountant sets the entries lock date to 2026-02-28. Posting an entry dated 2026-02-20 is refused with the lock date message of the ledger domain; posting an entry dated 2026-03-01 is accepted.
2. A lock date exception is granted to one user for the period up to 2026-03-05. That user posts the February entry; another user still cannot. The exception is recorded with its author, its reason and its validity, and revoking it restores the refusal.
3. Hashing is enabled on the journal INV. Every posted invoice receives a hash chained to the previous one. Attempting to modify the accounting date of a hashed entry is refused with the inalterability message.
4. The integrity report lists the first and last hashed entry of the journal, the chain state and, when a record has been altered outside the application, the first broken link.
5. Resetting a hashed entry to draft is refused; the only way back is a reversal.

---

## 18. Invariants checked continuously

These are not scenarios: they are assertions run after every scenario, over the whole fixture. Any violation fails the run that caused it.

| Invariant | Statement |
|---|---|
| INV-01 | For every posted Journal Entry, the sum of debits equals the sum of credits, exactly. |
| INV-02 | For every posted document, the sum of its line subtotals plus the sum of its tax amounts equals its total, exactly. |
| INV-03 | For every Journal Item, the company amount equals the currency amount converted at the entry's rate, rounded to the company currency step. |
| INV-04 | For every invoice, the residual equals the total minus the sum of the reconciled amounts, and never has a sign opposite to the document. |
| INV-05 | For every reconciled set of items, the sum of the reconciled amounts is zero in every currency involved, exchange differences included. |
| INV-06 | For every product under automated valuation, the balance of its stock valuation account equals the sum of its valuation layers. |
| INV-07 | For every product, and for every product and location pair, the quantity on hand equals the sum of its quantity records, and equals the sum of the signed quantities of its done movements. |
| INV-08 | For every product, the sum of the remaining quantities of its first in first out layers equals its quantity on hand when the quantity is positive. |
| INV-09 | For every completed manufacturing order, the production account balance attributable to it is zero, except for the released labour of an unbuild. |
| INV-10 | For every Journal Item with an analytic distribution, the sum of the analytic line amounts equals the item amount. |
| INV-11 | For every closed point of sale session, the session entry balances and the sum of its payment lines equals the sum of the order payments. |
| INV-12 | No document number is used twice inside one journal; every sequence has produced a strictly increasing series within each period; and the numbering has no gap that the gap check does not report. |
| INV-13 | No transfer is done while one of its moves is not done or cancelled. |
| INV-14 | No quantity record is negative in a location that forbids negative stock. |
| INV-15 | Every record that a record rule excludes from a user's queries is also excluded from that user's reports, exports and endpoint responses. |
| INV-16 | Every sent electronic mail, text message and letter has exactly one queue row and one final state. |
| INV-17 | Running every scheduled job twice in the same interval produces the effect once, unless the job is specified as repeating. |
| INV-18 | Reserved quantity never exceeds the quantity on hand at any location, and never falls below zero. |
| INV-19 | For every order line, the invoiced quantity never exceeds the delivered quantity where the policy is delivery-based, and never exceeds the ordered quantity where it is order-based, except by the documented upsell allowance. |
| INV-20 | Every record that declares a company is reachable only by a user whose company set contains that company. |
| INV-21 | For every document under a costing method that keeps layers, the sum of the values of the layers of a product equals the value the valuation statement reports for it. |

---

## 19. Tolerance rules

### 19.1 Amounts: no tolerance

There is no tolerance on any monetary amount, any quantity, any percentage, any rate or any count. A produced value either equals the expected value exactly, compared as a decimal, or the test fails. In particular:

1. Comparison uses decimal arithmetic. Comparing 0.1 + 0.2 with 0.3 must succeed; an implementation that compares binary floating point values will fail this plan somewhere, and that failure is a real defect.
2. "Close enough" is never accepted, not even for one unit in the last place, and not even for a report total.
3. An accumulated difference of one unit in the last place between a total and the sum of its parts is a failure, not a rounding artifact: the specification states, for every such total, whether it is computed from rounded parts or from unrounded parts.

### 19.2 Rounding: documented, never inferred

Every rounding is one of the following, and each test states which one it expects.

| Rounding | Rule | Where it applies |
|---|---|---|
| Currency rounding | `round_to(value, currency.rounding)`, ties away from zero | every monetary amount stored on a record or shown to a user |
| Quantity rounding | `round_to(value, unit.rounding)`, with the direction the caller declares (nearest, up or down) | every quantity on a document line, a movement or a production |
| Precision rounding | `round(value, decimal_places)` of the named decimal precision record | product prices, discounts, payment term percentages, analytic percentages, weights, volumes |
| No rounding | the value is kept at full precision and rounded only when it is finally stored or displayed | average costs, intermediate tax bases under the method "round globally", pricelist intermediate results, accrual fractions |
| Cash rounding | `round_to(total, cash_rounding.rounding)` with the method's direction (nearest, up, down) | the total of a document or a counter order that uses a cash rounding rule |

A test that does not know which rounding applies is an under-specified test: the specification is consulted, and if it is silent the gap is raised rather than guessed.

### 19.3 Dates and moments

| Value | Rule |
|---|---|
| A date field | Equality of the calendar date. No tolerance. |
| A moment field written by the system as "now" | The test freezes the clock; the expected value is the frozen moment. Where the clock cannot be frozen, the assertion is that the moment lies inside the run window, and the test states that explicitly. |
| A duration in days computed on a working schedule | Exact equality after the schedule's rounding, which the domain document states. |
| A due date | Exact equality of the calendar date, computed by the payment term arithmetic, including the end-of-month rule and the day-of-month rule. |

### 19.4 Text

| Value | Rule |
|---|---|
| A validation or error message | Exact equality, character for character, after removing any product name. Placeholders are substituted with the values the document shows. |
| A document label written by an operation (for example the label of a journal item) | Exact equality. |
| A rendered document's free text (terms, notes) | Exact equality of the text that the fixture supplied. |
| Translated text | Exact equality in the language the test selects; a missing translation falls back to the source language and the test asserts the fallback. |

### 19.5 Ordering

| Result | Rule |
|---|---|
| A list with a declared default order | Exact sequence equality. |
| A list with no declared order | Compared as a set. |
| The lines of a document | Exact sequence equality: the line sequence is part of the record. |
| The items of a journal entry | Compared as a multiset, unless the domain declares an order; the totals and the per-account aggregation are compared exactly. |
| Grouped read results | Exact sequence equality when the grouping declares an order, otherwise as a set of groups with exact values. |

---

## 20. Running the plan

### 20.1 Order of execution

1. The fixture is loaded into an empty system.
2. The unit rule tests run first: they are fast and they localize a failure to one rule.
3. The numeric precision tests run second: a rounding defect makes every later scenario fail for the wrong reason.
4. The state machine coverage tests run third.
5. The golden scenarios run in their numbered order, with the invariants of section 18 asserted after each one.
6. The accounting consequence tests run after the scenarios that produce their events, because each one replays an event and compares the complete item listing.
7. The access control matrices, the interface contract tests and the report content tests run last, on the state the scenarios left.

### 20.2 When each layer runs

Not every layer runs at every moment. The layers that are cheap and that localize a failure run constantly; the layers that need a built system run less often.

| When | Layers |
|---|---|
| On every change | One, two, five, nine |
| On every merge | One, two, three, four, five, seven, nine, ten |
| Nightly | All, including six, eight and eleven |
| Before a milestone or a stage gate | All, plus the differential harness of section 21 where a running reference is available |

A gate in [milestones](milestones.md) closes only when its layers pass with no unacknowledged difference. An acknowledged difference is one recorded in [coverage and evidence](coverage-and-evidence.md) with a reason and a decision; an unacknowledged one is a failure.

### 20.3 Isolation

Each scenario declares its predecessor, if any. A scenario with no predecessor runs on a freshly loaded fixture. A run of the whole plan therefore reloads the fixture between independent scenarios and keeps it between chained ones. No scenario depends on another scenario's surrogate identifiers: they refer to records by their business keys (document number, product name, account code, partner name).

### 20.4 What a failure report must contain

1. The scenario or rule identifier.
2. The step at which the divergence appeared.
3. The expected and the produced values, as decimals, with the field and the record named by its business key.
4. The rule identifiers the step exercises, from the coverage matrix, so that the defect can be attributed to a rule and not only to a scenario.
5. Whether any invariant of section 18 also broke, which distinguishes a local defect from a structural one.

### 20.5 Coverage reporting

After a full run, the report states, per domain: the number of rules covered and uncovered, the number of workflows executed, the number of state transitions exercised against the catalog total, the number of accounting events tested, the number of endpoints exercised, the number of reports rendered, and the list of acceptance criteria identifiers that did not run. A domain is complete when every one of those lists is empty.

The report also states four numbers per domain, and they are not interchangeable:

1. **Specified** — artifacts the specification describes.
2. **Implemented** — artifacts the rebuild provides.
3. **Exercised** — artifacts at least one test touches.
4. **Verified** — artifacts whose specified outcome a test asserts exactly.

The gap between exercised and verified is where false confidence lives: a test that creates an invoice exercises fifty fields and verifies six. [Coverage and evidence](coverage-and-evidence.md) carries these numbers for the specification itself and is the place where they are kept current.

---

## 21. Comparing against a running reference

Where a running instance of the described system is available, add a differential harness: perform the same operation on both systems, then compare the resulting records field by field, ignoring identifiers, timestamps and anything this plan marks as varying in section 19. This is the strongest evidence available, and it finds behaviors that no document captured.

Where it is not available, the worked examples, the golden scenarios and the acceptance criteria are the reference, and the coverage report must say so plainly. [Coverage and evidence](coverage-and-evidence.md) keeps the two kinds of evidence separate, because a rebuild verified only against the specification inherits every gap the specification has.

---

## Reconciliation notes

1. **Two taxonomies of test.** One version organized the suite into eight layers ordered by the class of divergence each catches; the other into eight categories ordered by the artifact each starts from. Six of the eight appear in both under different names and are merged: arithmetic with numeric precision, state machines with state machine coverage, business scenarios with workflow scenarios, contracts with interface contracts, authorization with the access control matrix, and invariants with the continuously checked invariants. The two that only the first version had — entity structure and concurrency and recovery — are kept as layers two and eight. The three that only the second version had — business rules, accounting consequences and report content — are kept as layers nine, ten and eleven. The numbering of the first eight is unchanged, because the stage gates of [milestones](milestones.md) cite layers by number.
2. **The invariants.** One version listed seventeen invariants, the other eight. Four statements of the shorter list had no counterpart and are added as INV-18 to INV-21: the reservation bound, the invoicing policy bound, the company reachability bound and the layer-to-statement agreement. Two statements of the longer list were widened rather than duplicated: INV-07 now asserts conservation per product and per product and location pair, and INV-12 now asserts that a sequence is strictly increasing within a period as well as free of unreported gaps.
3. **The data sets.** One version named three fixed data sets — the reference set, the configured set and the migration set — without specifying any of them; the other specified one fixture in full. Both are kept, and the specified fixture is the configured set. Section 3 says so in its first paragraph.
4. **When each layer runs.** The running order and the gating table come from the version that had them. They now cover the three added layers: layers nine and ten run at the same frequency as the layers whose failures they localize, and layer eleven runs nightly with the other layers that need a built system.
5. **The four reporting numbers.** Specified, implemented, exercised and verified come from the version that had them, and are folded into the coverage reporting of section 20.5 rather than kept as a separate section, because both versions asked for one report per domain.
6. **The differential harness.** The comparison against a running reference is kept as section 21 because [coverage and evidence](coverage-and-evidence.md) points a reader at this document for it.
7. **A citation of a file that is not part of a domain folder's standard set.** The subcontracting scenario cited a separate subcontracting file; it cites the workflows file of the manufacturing domain, which owns that procedure.
8. **Two counts were checked against the catalogs of this repository and one was corrected.** The state field catalog of this repository lists two hundred and fifty-two state fields, not the one hundred and eighty that one version cited, and layer three now says two hundred and fifty-two. Every seed count that the two versions share — countries, country groups, subdivisions, cities, banks, languages, currencies, units of measure, removal strategies, routes, tags, report structures, payment terms, delivery terms, cash rounding rules, barcode rules, chat bot steps, leave types, work entry types and vehicle brands — was re-counted from the reference data catalog and agrees exactly.
9. **No contradiction of fact was found between the two versions.** They describe the same arithmetic, the same rounding discipline and the same tolerance rule. The fixture's numbers were re-checked for internal consistency where the two versions overlap: the tax of the reference invoice, the balance of every entry shown in the golden scenarios, the counter session totals and the seven decimal precisions, which agree with the shipped precision catalog.
