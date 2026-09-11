# Accounting effects of the Projects and Tasks domain

**This domain produces no journal entries.**

No operation described anywhere in this folder — creating a project, moving a task through a
stage, closing a task, reaching a milestone, publishing a project update, collecting a customer
rating, sharing a project, instantiating a template — writes a journal entry, a journal item, a
tax line or a payment. There is no journal of its own, no sequence of accounting document numbers,
no posting state, no lock date, no reversal and no reconciliation.

This file explains what the domain does instead: which records it creates, changes or deletes that
cause **other** domains to post, and which figures it merely reads back from those domains without
writing anything.

---

## 1. Why there is nothing to post

An accounting event requires a measurable economic fact: a good or a service changing hands, an
obligation arising, a payment made. The facts this domain records are **planning and progress**
facts:

| Fact recorded here | Why it is not an accounting event |
|---|---|
| A task is created | No obligation arises. The plan may change or the task may be cancelled. |
| Allocated time is set on a task | Allocated time is an intention, not a measured consumption. Consumption is recorded by the timesheets domain. |
| A task moves from one stage to another | A stage is a workflow position, not a delivery. |
| A task is closed | Closing a task neither delivers a product nor triggers a billing obligation by itself. What triggers billing is the delivered quantity on a sales order item, which is maintained by the sales, timesheets and milestone mechanisms. |
| A milestone is reached | The milestone is a *marker*. Reaching it changes the delivered quantity on a sales order item, and it is the resulting **invoice** — created in the accounts-receivable domain — that posts. |
| A project update is published | A written status report has no economic content. |
| A customer rates a task | A satisfaction score has no economic content. |
| A person is added as a collaborator | An access grant has no economic content. |

The only monetary figures the domain shows — the whole profitability contract of
[calculations.md](calculations.md) §6 — are **read back** from documents owned by the sales,
accounts-receivable, accounts-payable, purchasing, analytic-accounting and timesheets domains. The
domain converts them, weights them by the analytic share and groups them; it never creates,
modifies or deletes any of them.

---

## 2. What this domain does create that leads other domains to post

### 2.1 The analytic account of a project

A project may carry an Analytic Account (`account.analytic.account`). When one is created from the
project, it is created with:

| Field of the analytic account | Value taken from the project |
|---|---|
| name | the project's name, or "Unknown Analytic Account" when the project has none |
| company | the project's company |
| customer | the project's customer |
| plan | the **project plan**, that is the first of the root analytic plans returned by the plan resolver |

Renaming the project renames the analytic account, but **only when exactly one project points at
that account**.

Deleting a project deletes its analytic account **only when that account has no analytic line**.
An analytic account may not be deleted while any task exists whose project points at it; the
refusal message is "Before we can bid farewell to these accounts, you need to tidy up the projects
linked to them by removing their existing tasks!"

Changing the company of a project whose analytic account carries analytic lines, or is shared with
another project, is refused: "The project's company cannot be changed if its analytic account has
analytic lines or if more than one project is linked to it." Otherwise the analytic account's
company follows the project's, falling back to the customer's.

**Accounting consequence.** The analytic account is the key on which journal items, purchase order
lines, sales order lines and timesheet lines are tagged. It is the *only* accounting artefact this
domain owns. Tagging itself — writing an analytic distribution onto a journal item — is performed
by the domain that owns the document, never here.

### 2.2 The milestone that advances a delivered quantity

With the sales-linked package installed, a Milestone may point at a Sales Order Item whose
delivered quantity is tracked by milestones. Ticking the milestone changes that item's delivered
quantity to

```formula
qty_delivered = ( Σ over reached milestones of the item of quantity_percentage ) × ordered_quantity
```

Changing a delivered quantity changes the item's invoicing status, which makes the amount
invoiceable. The invoice itself is created, numbered and posted by the accounts-receivable domain;
see [../accounts-receivable/README.md](../accounts-receivable/README.md). Nothing in this domain
touches the invoice.

The reverse operation — unticking a milestone — lowers the delivered quantity again, which can
turn an over-invoiced item into a credit-note candidate; again, the credit note is produced
elsewhere.

### 2.3 The task that a service sales order item creates

When a confirmed sales order line carries a service product configured to track its delivery
through a project or a task, the sales domain creates the Project and the Task. This domain
supplies the entities and their defaults; the sales domain decides when to create them. The
resulting task's link back to the sales order item is what later lets the timesheets domain
attribute recorded time to the right item.

### 2.4 The project's re-invoiced sales order

A project may nominate a Sales Order as its re-invoicing target. Operations elsewhere that are
configured to generate a re-invoiceable cost — a delivery whose operation type generates analytic
costs, a vendor bill line marked for re-invoicing, an expense — append a line to that order.
The appended line, and the invoice that later realises it, belong to the sales and
accounts-receivable domains.

### 2.5 Nothing else

No other record created by this domain has an accounting consequence. In particular:

- Task Stages, Project Stages, Project Tags, Project Roles, Project Collaborators, Project
  Updates, Task Recurrences, Personal Stage Assignments and Ratings have no accounting effect of
  any kind;
- allocated time is never valued;
- the working-time metrics are never valued;
- archiving or deleting a task never reverses anything.

---

## 3. What this domain reads back, and from where

| Profitability section | Read from | Owning domain |
|---|---|---|
| `other_revenues_aal`, `other_costs_aal` | analytic lines on the project's account with no journal item behind them | [analytic accounting](../analytic-accounting/README.md) |
| `service_revenues`, `materials`, the four `billable_*` sections, `downpayments` | sales order items, their untaxed invoiced and to-invoice amounts | [sales](../sales/README.md) |
| `other_invoice_revenues` | customer invoice and credit-note lines carrying the project's analytic account | [accounts receivable](../accounts-receivable/README.md) |
| `cost_of_goods_sold` | the cost-of-goods-sold journal items of those invoices, restricted to expense accounts | [inventory valuation and costing](../inventory-valuation-and-costing/README.md), realised on the invoice |
| `purchase_order` | confirmed purchase order lines carrying the project's analytic account, and the bill lines realising them | [purchasing](../purchasing/README.md) |
| `other_purchase_costs` | vendor bill and refund lines carrying the project's analytic account | [accounts payable](../accounts-payable/README.md) |
| the timesheet sections | timesheet analytic lines | [timesheets](../timesheets/README.md) |
| the project's invoice count and vendor bill count | journal items carrying the project's analytic account | general ledger |

Every one of those reads is performed with elevated rights so that a project manager who has no
accounting privilege can still see the totals, and every one applies the analytic share weighting
and the currency conversion described in [calculations.md](calculations.md) §6.

### 3.1 The sign convention

Because the contract adds revenues and costs to obtain a margin, the domain normalises the signs
of everything it reads:

| Source | Native sign | Transformation | Reported sign |
|---|---|---|---|
| customer invoice line | credit, so a negative balance | subtracted | positive (revenue) |
| customer credit-note line | debit, so a positive balance | subtracted | negative (reduces revenue) |
| cost-of-goods-sold journal item | debit, so a positive balance | subtracted | negative (cost) |
| vendor bill line | debit, so a positive balance | subtracted | negative (cost) |
| vendor refund line | credit, so a negative balance | subtracted | positive (reduces cost) |
| purchase order line subtotal | positive | subtracted | negative (cost) |
| analytic line | already signed: negative for a cost, positive for a revenue | used as is | unchanged |
| sales order item untaxed amount | positive | used as is | positive (revenue) |
| advance-invoice sales order item | positive invoiced amount | used as is for "invoiced", negated for "to invoice" | positive then negative |

A re-implementation must reproduce these signs exactly, because the margin and the four
percentages of [calculations.md](calculations.md) §6.11 depend on them.

### 3.2 Double-counting protections

Four exclusion mechanisms keep an amount from being reported twice. They must all be reproduced.

| # | Mechanism |
|---|---|
| 1 | Customer invoice lines attached to a sales order item that has already been counted in §6.5 are excluded from `other_invoice_revenues`. |
| 2 | Vendor bill lines attached to a purchase order line that has already been counted in `purchase_order` are excluded from `other_purchase_costs`. |
| 3 | Analytic lines that have a journal item behind them are excluded from `other_revenues_aal` and `other_costs_aal` — those amounts are already visible through the invoice and bill sections. With the purchasing package installed, timesheet-domain analytic lines whose journal item comes from a purchase order line are likewise excluded from the timesheet aggregation. |
| 4 | Timesheet analytic-line groups whose category is "vendor bill" are skipped, because the same amount is already reported by the product re-invoicing sections. |

A fifth, extensible mechanism exists: a list of invoice line identifiers "already claimed by
another profitability report". This domain leaves it empty; other domains may add to it.

---

## 4. Effects on other domains that are not accounting effects

For completeness, these are the remaining cross-domain consequences of operations in this domain.

| Operation here | Consequence elsewhere |
|---|---|
| A task is created, written, closed or deleted | Messages, notifications, tracked values and activity records are written in the messaging domain. |
| A stage carrying an electronic mail template is entered | An outgoing message is queued in the messaging domain. |
| A rating request is sent | A Rating record and an outgoing message are created; the customer's answer later posts a further message. |
| A project's visibility changes | Follower rows are created or deleted; portal access tokens are cleared. |
| A collaborator is added or removed | Follower rows change; the two dormant portal security records are activated or deactivated. |
| A project is archived or deleted | Every one of its tasks is archived or deleted, which cascades to personal stage assignments, ratings and messages. |
| A feature flag is switched | A privilege group is granted to, or revoked from, every internal user; two notification subtypes are hidden or shown. |
| A user account is created | Seven personal task stages are created for that user, and, with the personal to-do package, one welcome to-do. |
| The daily rating job runs | Outgoing messages are queued for every task of every project attached to a periodic rating stage. |
| A project's name changes | The linked analytic account is renamed, when exactly one project points at it. |
| A project's company changes | The linked analytic account's company changes with it, subject to the refusal rule of §2.1. |

None of these writes a journal entry.

---

## 5. What a re-implementation must therefore provide

1. An Analytic Account entity, or an equivalent cost-collection key, that a Project can own, that
   carries a company and a customer, that belongs to a plan, and that can be tagged on journal
   items, purchase order lines, sales order lines and time records with a percentage
   distribution.
2. A way to read, per analytic account: analytic lines with and without a journal item behind
   them; customer invoice and credit-note lines with their document state, their balance, their
   company currency, their date and their display kind; vendor bill and refund lines with the same;
   purchase order lines with their state, their subtotal, their currency and the bill lines
   realising them; sales order items with their state, their advance-invoice flag, their expense
   flag, their product, their currency, their untaxed invoiced amount, their untaxed to-invoice
   amount and their invoiced and to-invoice quantities.
3. A currency conversion able to convert an amount from one currency into another, in the context
   of a company, at a given date, with and without rounding to the target currency's precision.
4. Nothing else. No journal, no sequence, no posting, no reconciliation.
