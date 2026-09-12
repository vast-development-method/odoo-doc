# Learning, Questionnaires and Recognition — Accounting effects

## 1. This domain posts no journal entry

None of the thirty-three entities of this folder writes a journal entry, a journal item or any other
accounting record. A questionnaire, a participation, a course, a content item, an enrolment, a
forum, a post, a badge, a goal, a challenge, a rank and a reputation movement are all
non-financial records. The reputation-point balance is a community currency: it is a whole number
held on a user account, it has no monetary unit, no currency link and no company, it never appears
in any ledger and it is never revalued. A rebuild must not model it as money.

Three consequences follow and are worth stating plainly, because a reader coming from a financial
domain would look for them:

1. **No revenue is recognised when an attendee completes a course.** Completion moves reputation
   points and sends a message; it changes nothing in the ledger. A course paid for in advance is
   recognised by the selling domain at its own moment, not at the moment the learner finishes.
2. **No expense is recorded when a badge is granted.** A badge is a distinction, not a benefit in
   kind; if an organisation chooses to attach a cost to a badge, that cost is recorded outside this
   domain.
3. **No provision is made for an unexpired certification.** The validity period of a certification
   is an informational date on the employee's résumé line; it creates no liability and no accrual.

## 2. The one financial event the domain triggers

The domain triggers exactly one financial chain, and it triggers it entirely through another
domain's records: **selling access to a course**.

The chain is:

1. A course whose enrolment policy is `payment` points at a product whose service tracking is
   `course`. The product belongs to the
   [Products and catalog](../products-and-catalog/README.md) domain; this domain only adds the
   tracking value, the link back to the courses and the sales description `Access to: ` followed by
   the course names.
2. A customer puts that product on an order line. The line, its price, its taxes and its currency
   belong entirely to the [Sales](../sales/README.md) domain. This domain contributes two rules
   only: the line is forced to one unit ([`LSG-130`](business-rules.md#lsg-130)) and the product may
   always be added to the cart ([`LSG-131`](business-rules.md#lsg-131)).
3. Confirming the order enrols the customer. That is the **only** thing this domain does at
   confirmation: it creates a Course Enrolment. It writes nothing on the order, nothing on the
   invoice and nothing in the ledger.
4. The customer invoice derived from that order is created, taxed, posted and paid by the
   [Accounts receivable](../accounts-receivable/README.md) domain, and the resulting journal entry is
   specified by the [General ledger](../general-ledger/README.md) domain. The accounts are chosen by
   the product's income account, its product category, and the fiscal position of the customer,
   exactly as for any other service product; the tax treatment is the product's own; the currency
   and the rate are the order's; the counterparty is the customer; the analytic distribution is the
   one carried by the order line.

Nothing in this chain is specific to a course. A rebuild that already implements a service product
sold on an order needs no new ledger behaviour for courses.

## 3. Where to look for the entries

| Event | Domain that owns the ledger effect |
|---|---|
| Invoicing the sale of course access | [Accounts receivable](../accounts-receivable/README.md) for the customer invoice, [General ledger](../general-ledger/README.md) for the journal entry, [Taxes](../taxes/README.md) for the tax items. |
| Collecting the payment | [Payments and bank reconciliation](../payments-and-bank-reconciliation/README.md). |
| Recognising the revenue of a course sold in one currency and reported in another | [Multi-currency](../multi-currency/README.md). |
| Reporting the revenue per course | [Sales](../sales/README.md); the per-course revenue figure this domain shows is read from the confirmed sales-analysis amounts of the linked product and is a display figure, not a posting. |

## 4. The per-course revenue figure

The revenue shown on a course is the total of the confirmed sales-analysis amounts of its linked
product, expressed in the product's currency and visible only to the salesperson group. It is a
derived, unstored figure computed at read time. It aggregates **every** confirmed sale of that
product, so two courses sharing one product show the same total; a rebuild that needs revenue per
course must give each course its own product.

## 5. Industry-standard default

Nothing in the domain says what should happen to the revenue of a course when an attendee is removed
from it after failing the last attempt at a certification
([`LSG-134`](business-rules.md#lsg-134)). No credit note is created, no refund is proposed and no
record is written against the order.

**Industry-standard default:** the removal is a pedagogical consequence, not a commercial one. The
sale stands and the ledger is untouched; an organisation that wishes to refund such a learner issues
an ordinary credit note through the receivable domain, under its own policy. A rebuild should not
create any automatic financial consequence at that moment, because doing so would post an entry the
selling party never approved.
