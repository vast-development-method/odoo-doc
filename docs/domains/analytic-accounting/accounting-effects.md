# Analytic Accounting — Accounting Effects

This domain **posts nothing to the ledger**. It creates no journal entry, no journal item, no tax
line and no reconciliation, and it never changes the debit, the credit or the balance of an
existing journal item. Its entire relationship with the financial books runs in one direction: it
**reads** the journal items of posted entries and materialises their attribution as Analytic
Lines, which are management-accounting facts kept outside the double-entry books.

That statement is the first thing a rebuild must internalise, because it is the difference between
a parallel book and a second ledger. An analytic line has no counterpart, no debit column, no
credit column and no balancing invariant; two analytic lines of the same journal item are not
related to one another; a set of analytic lines never has to add up to zero.

This file specifies, in that order:

1. [The ledger boundary](#1-the-ledger-boundary)
2. [The sign convention](#2-the-sign-convention)
3. [The analytic record produced by a posting, item by item](#3-the-analytic-record-produced-by-a-posting-item-by-item)
4. [Event: post a journal entry](#4-event-post-a-journal-entry)
5. [Event: change the distribution of a posted journal item](#5-event-change-the-distribution-of-a-posted-journal-item)
6. [Event: reset a journal entry to draft, cancel it, delete it](#6-event-reset-a-journal-entry-to-draft-cancel-it-delete-it)
7. [Event: reverse a posted entry](#7-event-reverse-a-posted-entry)
8. [Event: reconcile, unreconcile, write off, apply a cash discount](#8-event-reconcile-unreconcile-write-off-apply-a-cash-discount)
9. [Event: post an entry expressed in a foreign currency](#9-event-post-an-entry-expressed-in-a-foreign-currency)
10. [Event: post an invoice with a granted discount booked to an allocation account](#10-event-post-an-invoice-with-a-granted-discount-booked-to-an-allocation-account)
11. [Event: confirm a source document that carries distributions](#11-event-confirm-a-source-document-that-carries-distributions)
12. [Event: value a stock move, a work order or a manufacturing order](#12-event-value-a-stock-move-a-work-order-or-a-manufacturing-order)
13. [Event: record a timesheet](#13-event-record-a-timesheet)
14. [Event: record an expense](#14-event-record-an-expense)
15. [Event: enter or edit an analytic line by hand](#15-event-enter-or-edit-an-analytic-line-by-hand)
16. [The ledger effects this domain triggers indirectly, and where they are specified](#16-the-ledger-effects-this-domain-triggers-indirectly-and-where-they-are-specified)
17. [Summary of the analytic effect of every accounting operation](#17-summary-of-the-analytic-effect-of-every-accounting-operation)
18. [Reconciliation notes](#18-reconciliation-notes)

---

## 1. The ledger boundary

| Question | Answer |
|---|---|
| Does this domain create journal entries? | No. Not one, in any situation. |
| Does this domain create or modify journal items? | It writes the field `analytic_distribution` (analytic distribution) on a journal item and reads that item's balance. It never writes the debit, the credit, the balance, the account, the currency, the date or the journal of a journal item. |
| Does this domain participate in the balance check of a journal entry? | No. Analytic lines are not part of the double-entry invariant. |
| Does this domain affect reconciliation? | No. Reconciling, unreconciling, writing off a residual amount and applying a cash discount all leave existing analytic lines untouched. The one interaction is described in section 8. |
| Does this domain affect the tax computation or the tax report? | No. A tax line inherits the distribution of the base lines it is computed on, which decides how the **tax amount** is attributed analytically; the tax amount itself, the tax grids and the tax report are unaffected. |
| Does this domain affect the trial balance, the balance sheet or the profit and loss statement? | No. Those are built from journal items. Analytic lines feed only the analytic reports and the profitability panels. |
| Can an analytic attribution change an amount posted to a financial account? | No. One case looks like an exception and is not: when a company books its granted discounts to a discount allocation account, the analytic distributions of the product lines decide how the **allocation line's distribution** is weighted. The amounts of the journal items are computed by the general ledger and are unaffected (section 10). |
| Does this domain block accounting operations? | Yes, in three places: posting is refused when an archived analytic account is named in a distribution ([business-rules.md](business-rules.md) `AA-078`); posting is refused when a mandatory plan is not fully distributed and the caller asked for the validation (`AA-079` to `AA-082`); and the confirmation of a sales order, a purchase order, an expense, a manufacturing order, a transfer or a timesheet is refused under the same mandatory plan rule (`AA-079`). |
| Does this domain produce a currency revaluation? | No. An analytic line carries one amount in one currency — the company currency of its own company — and no foreign-currency counterpart, so nothing about it can be revalued (`AA-069`). |
| Does an analytic line carry a tax? | No. It has no tax field, no tax grid and no tax base. |
| Is an analytic line ever reconciled? | No. There is no matching concept in the analytic book. |

**Why the domain owns no ledger entry.** Every amount an analytic line reports has already been
recognised in the financial books by the document that produced it: the vendor bill recognised the
cost, the customer invoice recognised the revenue, the expense entry recognised the reimbursement.
Recognising it a second time would double the books. The analytic line therefore restates an
amount already posted, along a different axis, and is deliberately kept out of the ledger.

---

## 2. The sign convention

An analytic line records a management fact, so it carries a single **signed** amount in the field
`amount` (amount) instead of a debit and a credit.

```formula
analytic_amount = − journal_item_balance × percentage ÷ 100

journal_item_balance = journal_item_debit − journal_item_credit        (in the company currency)
```

| Journal item | Balance | Analytic amount at one hundred percent | Reading |
|---|---|---|---|
| Expense account debited by 1 000.00 | +1 000.00 | −1 000.00 | A cost charged to the axis value |
| Income account credited by 2 400.00 | −2 400.00 | +2 400.00 | Revenue attributed to the axis value |
| Fixed asset account debited by 5 000.00 | +5 000.00 | −5 000.00 | An investment charged to the axis value |
| Income account debited by 300.00, a customer credit note | +300.00 | −300.00 | Revenue reversed |
| Expense account credited by 200.00, a supplier credit note | −200.00 | +200.00 | A cost reduction |

From those signed amounts the analytic account derives three figures, specified in
[calculations.md](calculations.md) section 13:

```formula
credit_of_account  = Σ of the amounts that are ≥ 0
debit_of_account   = − Σ of the amounts that are < 0
balance_of_account = credit_of_account − debit_of_account
```

The balance is presented on the account's form under the label "Gross Margin". Note the deliberate
mirror with the financial book: what is a **debit** in the ledger becomes a **negative** analytic
amount and is reported as **debit** — cost — on the analytic account; what is a **credit** in the
ledger becomes a **positive** analytic amount and is reported as **credit** — revenue. The two
books agree in meaning while using opposite signs, and a rebuild that omits the negation produces
an analytic book in which every project appears to earn its costs.

---

## 3. The analytic record produced by a posting, item by item

An accounting specification itemises a ledger entry. This domain produces no ledger entry, so the
equivalent itemisation is of the analytic line. For each analytic line created by a posting, the
values are fixed as follows. The complete arithmetic is in [calculations.md](calculations.md)
sections 9 and 10.

| Item of the analytic line | Identifier | Rule that fixes it |
|---|---|---|
| Description | `name` | The journal item's label; when that is empty, the journal item's reference; when both are empty, the text formed by a solidus, the text " -- " and either the partner's name or a solidus. |
| Date | `date` | The journal item's accounting date, **after** the shift the lock dates may have applied during posting (`AA-094`). Never the document date, never today. |
| Analytic account of each axis | `account_id` for the base plan, `x_plan<plan identifier>_id` for every other root plan | One account per account named in the distribution key, each written into the column of that account's **own root plan**. A key naming two accounts of two axes produces **one** line carrying both. |
| Amount | `amount` | − journal item balance × percentage ÷ 100, with the closing-line rule of `AA-088` and the rounding-error cancellation of `AA-090`, rounded onto the **company** currency's rounding step. |
| Currency | `currency_id` | Derived from the company's currency. An analytic line has no document currency and no rate of its own. |
| Quantity | `unit_amount` | The journal item's quantity, **copied whole onto every slice** and never divided by the percentage (`AA-108`). |
| Unit | `product_uom_id` | The journal item's unit of measure. |
| Product | `product_id` | The journal item's product. |
| Financial account | `general_account_id` | The journal item's account. The constraint `AA-067` keeps the two equal for as long as the line points at that item. |
| Financial journal | `journal_id` | Derived from the journal item's journal. |
| Journal item | `move_line_id` | The journal item itself. Deleting that item deletes the analytic line by cascade (`AA-074`). |
| Reference | `ref` | The journal item's reference. |
| Partner | `partner_id` | The journal item's partner; when the journal item has none, the value already stored is kept. |
| User | `user_id` | The entry's invoicing salesperson when set, otherwise the acting user. |
| Company | `company_id` | The journal item's company, or the company the reader is acting for when the item has none. Set once and never changed (`AA-071`). |
| Category | `category` | `invoice` for a sale document, `vendor_bill` for a purchase document, `other` for anything else. |

**Which journal items produce analytic lines.** Every journal item of a posted entry that carries a
non-empty distribution, whatever its display type:

| Journal item kind | Carries a distribution? | Where the distribution comes from |
|---|---|---|
| Product line of an invoice, a bill or a receipt | yes | Typed by the reader, proposed by a distribution model, or inherited from the linked sales order line or purchase order line. |
| Line of a miscellaneous entry | yes when typed | Typed by the reader, or proposed by a distribution model. |
| Tax line | yes | Copied from the base lines it is computed on, through the grouping key used to build tax lines. |
| Early payment discount line | yes | Copied from the base line; its counterpart on the cash discount account takes the distribution proposed by the distribution models with the cash discount account's code as the account prefix. |
| Discount allocation line | yes | Weighted from the distributions of the product lines whose discounts it allocates, as computed in [calculations.md](calculations.md) section 15. |
| Payment term line, receivable or payable | no | Never carries one, therefore never produces an analytic line. |
| Section, subsection and note lines | no | They carry no amount. |
| Exchange difference line | only when the caller supplies one | See section 8. |

**Postcondition of a posting.** For each journal item and each root plan whose distribution totals
exactly one hundred percent, the amounts of the analytic lines of that journal item add up exactly
to the negated balance of the journal item. For a plan whose total is below or above one hundred,
they add up to the negated balance times that total divided by one hundred.

---

## 4. Event: post a journal entry

**Trigger.** Any journal entry moves to the posted state: a customer invoice, a credit note, a
vendor bill, a refund, a receipt, a miscellaneous entry, an opening entry, a payment entry, a
cash-basis entry, an adjusting entry, an exchange difference entry, an entry created by the
automatic posting job or by a recurring template.

**Guards, in the order they run.**

1. Every analytic account named in the distribution of every journal item of the entries being
   posted is read with elevated rights and with archived records included; when one of them is
   archived the posting is refused with "You cannot post an entry with an archived analytic
   account: the account names", the placeholder listing the names of every archived account found,
   separated by a comma and a space (`AA-078`).
2. The accounting date of each entry is adjusted for the violated lock dates.
3. When the caller switched the validation flag on, every journal item whose display type is
   `product` is validated against the mandatory plans; a failure refuses the posting with "One or
   more lines require a 100% analytic distribution." (`AA-081`), with the mass-posting redirect of
   `AA-082` when more than one entry is being posted.

**Journal entries generated by this domain.** None.

**Analytic records written.** One Analytic Line per distribution entry of every journal item that
carries a non-empty distribution, minus the entries whose amount passes the zero test at the
company currency's rounding step (`AA-087`). The values are those of section 3. All the lines of
all the entries being posted are created in one operation, with the synchronisation guard raised
(`AA-092`).

**Worked example.** A vendor bill of the company *Northwind*, whose currency rounds onto one
hundredth, has one product line of 1 000.00 booked on the expense account `600000`, with the
distribution `{ "7,12" : 60 , "8,12" : 40 }`: accounts 7 (*Research and Development*) and 8
(*Administration*) belong to the root plan *Departments*, account 12 (*Project Alpha*) to the root
plan *Project*.

The journal entry the general ledger posts, unchanged by this domain:

| Financial account | Debit | Credit |
|---|---|---|
| `600000` Expenses | 1 000.00 | |
| `400000` Accounts payable | | 1 000.00 |

The analytic lines this domain writes:

| `account_id` (*Project*) | *Departments* column | `amount` | `unit_amount` | `general_account_id` | `category` |
|---|---|---|---|---|---|
| Project Alpha | Research and Development | −600.00 | 1 | `600000` | `vendor_bill` |
| Project Alpha | Administration | −400.00 | 1 | `600000` | `vendor_bill` |

Effect on the analytic accounts: *Research and Development* gains 600.00 of debit, *Administration*
400.00 of debit, *Project Alpha* 1 000.00 of debit. The payable journal item carries no
distribution and produces nothing. The financial books are untouched: the entry still shows
1 000.00 of expense and 1 000.00 of payable.

**The two axes do not double-count.** The two lines total −1 000.00, which is the whole cost. Read
along *Departments* they split it 600 and 400; read along *Project* they aggregate to 1 000. A
rebuild that adds the analytic lines across axes reports two thousand of cost and is wrong.

---

## 5. Event: change the distribution of a posted journal item

**Trigger.** A reader or an integration writes `analytic_distribution` on a journal item whose
entry is posted, from the distribution editor, from the mass-edit of a list, or programmatically.

**Journal entries generated.** None. The journal item's debit, credit, balance, account, date and
journal are untouched, and no reversal entry is created.

**Analytic records written.** The stored distribution is read straight from storage, merged with
the incoming document by the algorithm of [calculations.md](calculations.md) section 7, and stored;
then, for the journal items whose entry is posted, **all** their analytic lines are deleted and
re-created from the merged distribution, which re-runs the mandatory plan validation, the
closing-line rule and the rounding-error cancellation. The identifiers of the analytic lines
therefore change (`AA-091`). Journal items of a draft entry receive the merged distribution and no
analytic line at all (`AA-073`).

**Worked example.** The bill of section 4 is corrected to `{ "7,12" : 100 }`. The two analytic
lines are deleted and one line of −1 000.00 carrying *Research and Development* and *Project
Alpha* is created. *Administration* loses its 400.00 of debit; *Project Alpha* keeps 1 000.00.

---

## 6. Event: reset a journal entry to draft, cancel it, delete it

| Operation | Analytic effect |
|---|---|
| Reset to draft | Every analytic line of every journal item of the entry is deleted, with the synchronisation guard raised, so that the deletion does not rewrite the distributions. The distributions stored on the journal items survive untouched, therefore posting the entry again re-creates equivalent lines with new identifiers (`AA-074`, `AA-092`). |
| Cancel | The cancellation path of the general ledger reaches the draft state through the same reset, so the analytic effect is the reset effect. There is no separate analytic state for a cancelled entry. |
| Delete a journal item | Its analytic lines are deleted by cascade, because the link `move_line_id` carries a cascading deletion rule (`AA-074`). |
| Delete a journal entry | Its journal items and therefore all their analytic lines are deleted by cascade. |

**No trace is left.** A reset to draft leaves nothing in the analytic book: no reversing line, no
audit record of the deleted lines. A rebuild that needs an audit trail of analytic movements must
use the reversal path of section 7, which is what the accounting practice expects for a posted
period.

---

## 7. Event: reverse a posted entry

**Trigger.** The general ledger creates a reversing entry — a credit note, a refund or a plain
reversal — either as a draft or posted immediately.

**Journal entries generated.** By the general ledger only; they are specified in
[../general-ledger/accounting-effects.md](../general-ledger/accounting-effects.md).

**Analytic effect.** The reversing entry's journal items carry the **copied** distribution of the
items they reverse, because `analytic_distribution` is a copied field. When the reversing entry is
posted it goes through the ordinary generation of section 4. Because every balance is negated,
every analytic amount is negated:

```formula
reversing_analytic_amount = − ( − original_balance ) × percentage ÷ 100 = − original_analytic_amount
```

The original analytic lines are **kept**. Both movements stay visible with their own dates, so a
balance computed over a range that contains only the original date still shows the original cost.

**Worked example.** The bill of section 4, dated 1 March 2026, is refunded in full on 31 March
2026. The refund's product line credits `600000` by 1 000.00, a balance of −1 000.00, with the same
distribution. Posting it produces +600.00 on (*Project Alpha*, *Research and Development*) and
+400.00 on (*Project Alpha*, *Administration*).

| Analytic account | Lines | Total |
|---|---|---|
| Research and Development | −600.00 on 1 March, +600.00 on 31 March | 0.00 |
| Administration | −400.00 on 1 March, +400.00 on 31 March | 0.00 |
| Project Alpha | −1 000.00 on 1 March, +1 000.00 on 31 March | 0.00 |

Each account now shows 600.00, 400.00 and 1 000.00 of **both** debit and credit and a balance of
zero: the analytic book records the gross movements, not the net.

---

## 8. Event: reconcile, unreconcile, write off, apply a cash discount

**Analytic effect: none.** Matching two journal items, breaking a match, writing off a residual
amount and applying a cash discount at payment time do not change the balance of an already posted
journal item, so no analytic line is created, changed or deleted.

**The one interaction — exchange difference entries.** When a reconciliation between amounts in a
foreign currency produces an exchange difference entry, that entry is posted with the validation
flag explicitly switched **off**, therefore a mandatory plan never blocks a reconciliation
(`AA-085`). Its journal items carry a distribution only when the caller supplies one; when they do,
posting them generates analytic lines exactly like any other posted item, and the exchange gain or
loss is then attributed to the axis values named in that distribution.

**Cash discount lines.** The early payment discount lines that a payment term with a discount
produces at **invoicing** time are ordinary journal items of the invoice: the line on the product's
own account copies the base line's distribution, and the counterpart on the cash discount account
takes the distribution proposed by the distribution models with that account's code as the account
prefix, the company, the commercial partner and the partner's categories. Both are posted with the
invoice and produce analytic lines through section 4.

---

## 9. Event: post an entry expressed in a foreign currency

**Analytic effect.** The analytic amount is always derived from the journal item's **balance**,
which the general ledger has already converted into the company currency, and it is rounded with
the **company** currency's rounding step (`AA-069`, `AA-102`). An analytic line has no
foreign-currency amount, no rate and no revaluation.

**Worked example one.** An invoice of 10.00 in a currency whose rounding step is one unit, at three
units of that currency per unit of company currency. The journal item's balance is
10 ÷ 3 = 3.3333, which the general ledger rounds to 3.33. With the distribution `{ "1" : 100 }`,
which closes the plan, the analytic amount is − ( −3.33 ) × 100 ÷ 100 = 3.33, rounded with the
company step of 0.01 to **3.33**. Rounding with the document currency's step of one unit would have
produced 3, which would not tie back to the journal item.

**Worked example two.** An invoice of 2.00 in a currency whose rounding step is one unit, at one
hundred units per unit of company currency. The balance is 2 ÷ 100 = 0.02 and the analytic line is
**0.02**, not zero, because the zero test that discards a candidate line also uses the company
currency's step.

**Cross-currency reporting.** When one analytic account carries lines from companies with different
currencies, the account's debit, credit and balance convert each currency group at **today's** rate
for the company the reader is acting for, not at the rate of the line's date (`AA-101`). The figure
therefore changes from one day to the next. This is current behaviour and is reproduced
deliberately; the worked example is in [calculations.md](calculations.md) section 13.4, case three.

---

## 10. Event: post an invoice with a granted discount booked to an allocation account

**Trigger.** The company has set a discount allocation account and at least one invoice line
carries a discount percentage.

**Journal entries generated.** By the general ledger: one discount journal item on the product
line's own income account for the negated discount, and one counterpart journal item on the
discount allocation account for the total discount of the group made of the invoice, the account
and the rate.

**Analytic effect of this domain.** The discount journal item on the product line's own account
inherits that product line's distribution unchanged. The counterpart on the allocation account
receives a distribution weighted by the discount amounts, as computed in
[calculations.md](calculations.md) section 15. Both then produce analytic lines through section 4.

**Worked example.** Two untaxed invoice lines of 200.00 each; the first discounted twenty percent
with the distribution `{ "1" : 100 }`, the second discounted ten percent with `{ "2" : 100 }`; the
company's discount allocation account is `DIS`.

| Journal item | Financial account | Debit | Credit | Distribution | Analytic lines produced |
|---|---|---|---|---|---|
| Product line one | income of product A | | 160.00 | `{ "1" : 100 }` | +160.00 on account 1 |
| Product line two | income of product B | | 180.00 | `{ "2" : 100 }` | +180.00 on account 2 |
| Discount of line one | income of product A | | 40.00 | `{ "1" : 100 }` | +40.00 on account 1 |
| Discount allocation | `DIS` | 60.00 | | `{ "1" : 66.67 , "2" : 33.33 }` | −40.00 on account 1 and −20.00 on account 2 |
| Discount of line two | income of product B | | 20.00 | `{ "2" : 100 }` | +20.00 on account 2 |
| Payment term line | receivable | 340.00 | | empty | none |

The allocation line's two analytic amounts are − 60 × 66.67 ÷ 100 = −40.002, rounded to −40.00, and
− 60 × 33.33 ÷ 100 = −19.998, rounded to −20.00; the accumulated rounding drift is
+0.002 − 0.002 = 0.000, so the cancelling pass of `AA-090` exits at its first test. Account 1 nets
160.00 + 40.00 − 40.00 = 160.00 and account 2 nets 180.00 + 20.00 − 20.00 = 180.00, which are the
revenues actually earned after discount.

---

## 11. Event: confirm a source document that carries distributions

**Trigger.** A sales order, a purchase order, an expense, a manufacturing order, a warehouse
transfer or a timesheet is confirmed, approved or validated from the interface.

**Journal entries generated by this domain.** None. The confirmation may generate journal entries
through the owning domain; those entries produce analytic lines through section 4.

**Analytic effect.** The mandatory plan validation runs on each line of the document with the
business domain of that document, and a failure aborts the confirmation with "One or more lines
require a 100% analytic distribution." (`AA-079`, workflow 27 of [workflows.md](workflows.md)).
Confirming also freezes the distribution that will later be **copied** onto the invoice or bill
lines created from the document, which is how a distribution typed on an order reaches the ledger.

---

## 12. Event: value a stock move, a work order or a manufacturing order

**Trigger.** An inventory or manufacturing document recomputes its cost.

**Analytic effect.** The redistribution of [calculations.md](calculations.md) section 16 updates the
analytic lines already attached to the document **in place**: a line whose combination of accounts
is still a key of the new distribution is rewritten with the new amount and the new quantity; a
line whose combination has disappeared is deleted; a line whose new amount rounds to zero is
deleted; a combination with no line yet produces a new line. Unlike the regeneration of section 5,
this operation **preserves the identifiers** of the surviving lines, which is what lets a valuation
document keep pointing at its own analytic lines across successive costings.

The triggering events, the amounts and the quantities belong to
[../inventory-valuation-and-costing/](../inventory-valuation-and-costing/README.md) and
[../manufacturing/](../manufacturing/README.md); the journal entries those domains post are
specified there, and each of them produces analytic lines through section 4 as well when its
journal items carry a distribution.

---

## 13. Event: record a timesheet

**Analytic effect.** A timesheet line **is** an analytic line: it carries the same plan columns,
the same signed amount and the same quantity, with the quantity expressed in a time unit and the
amount derived from the employee's hourly cost, therefore negative or zero. Its base plan account
comes from the project; the mandatory plans other than the base plan must be filled explicitly
(`AA-079`, workflow 27). A timesheet analytic line has **no** journal item, so it contributes to
the analytic account's debit without any counterpart in the ledger, and it is one of the lines the
project profitability sections of [calculations.md](calculations.md) section 17 read. The employee
cost formula, the units and the billing consequences belong to
[../timesheets/](../timesheets/README.md).

---

## 14. Event: record an expense

**Analytic effect.** An expense carries a distribution of its own. Approving it runs the mandatory
plan validation with the business domain `expense`; posting the expense entry copies the
distribution onto the journal items it creates, and those journal items produce analytic lines
through section 4. An analytic account named in the distribution of an expense cannot be deleted
(`AA-033`). The expense entry itself, its accounts and its journals belong to
[../expenses/](../expenses/README.md).

---

## 15. Event: enter or edit an analytic line by hand

**Analytic effect.** A line entered by hand from the analytic items screen has no journal item and
produces no ledger effect whatsoever; it contributes to the analytic account's debit, credit and
balance exactly like a generated line. When the line **does** point at a journal item, creating,
editing or deleting it rebuilds that journal item's distribution from its remaining analytic lines
(`AA-093`, [calculations.md](calculations.md) section 11) — the distribution follows the lines
rather than the lines following the distribution. The rebuild writes with the synchronisation guard
raised, so the hand-edited lines are not deleted and regenerated, and the ledger is again
untouched: the journal item keeps its debit, its credit and its balance.

A consequence a rebuild must accept: after a manual edit the distribution of a posted journal item
may total more or less than one hundred percent, and the system records it as it stands. The next
posting of that document — after a reset to draft — would then be refused if a plan is mandatory.

---

## 16. The ledger effects this domain triggers indirectly, and where they are specified

The domain produces no ledger effect of its own, but it participates in the events of the domains
below, which do. Each row states what this domain contributes and where the ledger entry itself is
itemised.

| Domain | Ledger event | What this domain contributes |
|---|---|---|
| [../general-ledger/accounting-effects.md](../general-ledger/accounting-effects.md) | Posting any entry | The archived-account guard, the mandatory plan validation, the creation of the analytic lines and their deletion on a reset to draft. |
| [../accounts-receivable/accounting-effects.md](../accounts-receivable/accounting-effects.md) | Posting a customer invoice or credit note | The distribution defaulted on the invoice lines, the distribution inherited by the tax and discount lines, and the analytic lines produced with the business domain `invoice`. |
| [../accounts-payable/accounting-effects.md](../accounts-payable/accounting-effects.md) | Posting a vendor bill or refund | The same with the business domain `bill`. |
| [../sales/accounting-effects.md](../sales/accounting-effects.md) | Invoicing a sales order | The distribution carried from the order line to the invoice line, and the mandatory plan check at confirmation with the business domain `sale_order`. |
| [../purchasing/accounting-effects.md](../purchasing/accounting-effects.md) | Billing a purchase order | The same with the business domain `purchase_order`. |
| [../expenses/accounting-effects.md](../expenses/accounting-effects.md) | Posting an expense entry | The distribution on the expense and the check at approval with the business domain `expense`. |
| [../timesheets/accounting-effects.md](../timesheets/accounting-effects.md) | None in the ledger; the timesheet cost stays outside it | The analytic line itself, which **is** the timesheet. |
| [../inventory-valuation-and-costing/accounting-effects.md](../inventory-valuation-and-costing/accounting-effects.md) | Posting a valuation entry | The distribution on the valuation document and the in-place redistribution of its analytic lines. |
| [../manufacturing/accounting-effects.md](../manufacturing/accounting-effects.md) | Posting a production cost entry | The distribution on the work centre and the analytic lines of category `manufacturing_order`. |
| [../inventory-operations/accounting-effects.md](../inventory-operations/accounting-effects.md) | Validating a transfer | The mandatory plan check with the business domain `stock_picking` and the analytic lines of category `picking_entry`. |
| [../payments-and-bank-reconciliation/accounting-effects.md](../payments-and-bank-reconciliation/accounting-effects.md) | Posting an entry from a reconciliation model, posting an exchange difference | The distribution carried by a reconciliation model line, and the validation flag switched off on the exchange difference entry. |
| [../taxes/accounting-effects.md](../taxes/accounting-effects.md) | Posting the tax lines of a document | The distribution copied onto each tax line from its base lines, which decides how the tax amount is attributed. |
| [../financial-reporting/README.md](../financial-reporting/README.md) | None | The analytic filter and the analytic grouping of the financial reports. |

---

## 17. Summary of the analytic effect of every accounting operation

| Operation on the accounting side | Effect on analytic lines |
|---|---|
| Post an entry | Analytic lines are created from the distributions of its journal items. |
| Post an entry with an archived account in a distribution | Refused; nothing is created and the entry stays draft. |
| Post an entry whose mandatory plan is incomplete, with the validation flag on | Refused; nothing is created. |
| Post the same entry with the validation flag off | Posted; the analytic lines are created from the incomplete distribution. |
| Reset an entry to draft | All its analytic lines are deleted; the distributions survive. |
| Post it again | Equivalent analytic lines are created again, with new identifiers. |
| Cancel an entry | The reset path applies; the lines are deleted the same way. |
| Reverse an entry | The reversing entry copies the distributions and, on posting, produces opposite lines; the original lines are kept. |
| Change a posted journal item's distribution | Its analytic lines are deleted and regenerated with new identifiers. |
| Change a draft journal item's distribution | Only the distribution is stored; no analytic line exists to change. |
| Delete a journal item | Its analytic lines are deleted by cascade. |
| Delete a journal entry | The same, through its journal items. |
| Reconcile or unreconcile two items | Nothing. |
| Write off a residual amount | Nothing beyond the analytic lines the write-off entry itself produces. |
| Post an exchange difference entry | Posted without the mandatory plan check; analytic lines only when its items carry a distribution. |
| Archive an analytic account | Nothing changes for existing lines; future postings that name it are refused. |
| Delete an analytic account | Refused while any analytic line still holds it in a plan column; a distribution that merely names it does not block the deletion and the identifier is then ignored. |
| Recompute the cost of a valuation document | Its analytic lines are updated in place, keeping their identifiers. |
| Create, edit or delete an analytic line by hand | The journal item's distribution is rebuilt from its lines; the ledger is untouched. |
| Change the company of an analytic account | Refused while lines outside the new company's sub-tree carry it; nothing is posted either way. |

---

## 18. Reconciliation notes

1. **A single source.** Only one of the two drafts of this folder carried an accounting-effects
   document. It is kept in full here, with three corrections and several additions.
2. **Field identifiers.** The former draft named the fields of a generated analytic line by
   invented names — a "partner", a "financial account", a "unit of measure", a "journal item" and a
   "user" column. The stored names are contractual, so section 3 reproduces `partner_id`,
   `general_account_id`, `product_uom_id`, `move_line_id`, `user_id`, `unit_amount`, `ref`,
   `category` and the plan columns `account_id` and `x_plan<plan identifier>_id`, each with its full
   name in words, exactly as [entities.md](entities.md) does.
3. **The default description of a generated line.** The former draft read the fallback as the
   reference followed by the partner's name. The composition binds the solidus to the separator, so
   a journal item with a reference and no label yields the **reference alone**; section 3 states the
   corrected rule, which matches [calculations.md](calculations.md) section 9.3.
4. **The base plan column.** The former draft called the fixed column of the base plan a "project
   plan account" column. Its reproduced name is `account_id`.
5. **Events added.** The former draft had no section for an expense, for a manual analytic line or
   for the indirect ledger effects of the neighbouring domains. Sections 14, 15 and 16 are new,
   written from the source, because a reader of this file has to know where the ledger entries
   actually are.
6. **Reversal versus reset.** Both drafts described the two operations; this file states in section
   6 and section 7 why they differ — the first leaves no trace in the analytic book and the second
   leaves two movements — because that distinction is the most common rebuild error in this area.
