# Expenses — Glossary

Every term this folder uses with a precise meaning, defined. A term whose meaning is set by another
domain is defined here only as this domain uses it, with a link to the folder that owns it.

Reproduced identifiers appear in code font. Where a term has a stored value behind it, that value
is given, because it is what an integration reads.

---

## A

**Accounting date.** The date at which an entry takes effect in the ledger. For an **employee-paid**
expense it is derived from the bill date chosen in the posting dialogue by the general rule of
[`../general-ledger/`](../general-ledger/): a bill date in a month earlier than the current one, on
a journal whose numbering series resets monthly, is pushed to the last day of the bill month;
otherwise the accounting date is the later of the bill date and today, pushed further by any lock
date. For a **company-paid** expense it is simply the expense date, because a miscellaneous entry
is not an invoice. See [`accounting-effects.md`](accounting-effects.md) §3.1 and §4.2.

**Acknowledgement.** The message the system sends back after turning an electronic mail message
into an expense. Rendered from one of two templates according to whether the employee has a user,
it states the category (or that none was recognised), the **unit price** and the currency symbol,
and offers a link to the expense. See [`configuration.md`](configuration.md) §13.

**Administrator (expenses).** The highest of the three expense security levels. Sees and approves
every expense of the companies in the reader's allowed set, may configure expense categories and
activity types, and is exempt from the "not from your department" and "your own expense" approval
refusals. Stored group identifier `group_hr_expense_manager`, labelled *Administrator*. See
[`configuration.md`](configuration.md) §6.

**All Approver.** The middle expense security level. Approves any expense of the companies in the
reader's allowed set but does not configure the domain. Stored group identifier
`group_hr_expense_user`, labelled *All Approver*; it implies Team Approver.

**Alias.** The mailbox record that turns incoming electronic mail into expenses. Its local part is
`expense` as shipped, its contact policy is `employees`, and its target entity is the Expense. See
[`configuration.md`](configuration.md) §10.

**Amount-driven pricing.** The pricing model used when the expense category has **no** unit cost:
the employee types the total in the receipt currency and the system derives the unit price, leaving
the quantity at one. Contrast **quantity-driven pricing**. See
[`calculations.md`](calculations.md) §2.1.

**Analytic account.** A cost-carrying dimension — a project, a department, a customer contract —
against which an expense is charged. An analytic account named by an expense's distribution may
not be deleted. Owned by [`../analytic-accounting/`](../analytic-accounting/).

**Analytic distribution.** A mapping from analytic account to percentage, held on the expense and
copied onto the product line of its journal entry. Percentages must total one hundred per plan. A
plan may be declared **mandatory** for the business domain `expense`, in which case the approval
operation validates the distribution against it.

**Analytic line.** The cost record created for each analytic account named in a journal item's
distribution when that item is posted. Its amount is the negated balance apportioned by the
percentage, and its unit amount is the journal item's quantity. Rebilling rides on these lines.

**Approval activity.** A scheduled task of the type *Expense Approval*, created on the responsible
approver when an expense is submitted, marked **done** when it is approved, and **removed** when it
is refused or reset. It is the only nudge the approver receives at submission time; no message is
sent then.

**Approval date.** The moment an expense was approved. Written by the approval operation, never
cleared by refusal, cleared by reset. Storage name `approval_date`.

**Approval state.** The hidden driver of the visible status that records how far the human approval
chain has got. Its values are empty, `submitted`, `approved` and `refused`. It is read-only to
users and is never copied when an expense is duplicated. Storage name `approval_state`. See
[`state-machines.md`](state-machines.md) §3.

**Approver.** Any user who may approve a given expense. Which users those are is decided by the
four-branch test of [`business-rules.md`](business-rules.md) §3.3, not by a single field.

**At cost.** One of the three rebilling policies. The rebilling line's unit price is the expense's
**untaxed** cost per unit, taken from the journal item as
`abs((credit − debit) ÷ item quantity)`. Stored value `cost`.

**At sales price.** One of the three rebilling policies. The rebilling line's unit price is the
category's sales price for a quantity of one at the order date. Stored value `sales_price`.

**Attachment.** A file held against an expense: the receipt. Attachments may be **added** up to
approval and **deleted** up to submission. Copies of every attachment are made onto the journal
entry when the expense is posted.

**Automatic validation.** The branch of the submit operation that skips the *Submitted* status
entirely and approves the expense at once. It applies when the expense has no manager **and** the
employee has no designated expense approver, or when the expense's manager is the employee's own
user. The duplicate check is deliberately not run on this path; the mandatory-plan validation is.

---

## B

**Bill date.** The date of the underlying document. On an employee-paid receipt it is the
accounting date chosen in the posting dialogue; a company-paid entry has none, being a
miscellaneous entry. A reversal produced by resetting an expense has its bill date forced to
**today**.

**Branch company.** A company that is a subdivision of another and keeps its own books. An expense
may belong to a branch; its entry is then written in the branch's books with the branch's journal
and its currency.

---

## C

**Can be expensed.** The flag on a Product Variant that makes it selectable as an expense category.
Storage name `can_be_expensed`. It cannot disagree with the purchasable flag: setting one sets the
other.

**Can be rebilled.** The derived flag on an expense that is true when its category's rebilling
policy is *at cost* or *at sales price*. When it turns false the sales order reference is cleared.

**Cash-basis tax.** A tax that becomes exigible when the money moves rather than when the document
is issued. On the **employee-paid** path its line lands on the tax's cash-basis transition account
and carries no tax report tags until the receipt is paid. On the **company-paid** path it lands on
the tax's final account and carries its tags immediately, because the money has already left.

**Category.** See **expense category**.

**Commercial partner.** The contact an accounting document is ultimately billed to or from. For an
expense entry it is the employee's user partner, computed by the expense-specific rule that stops
an employee whose contact hangs under the company's own contact record from being billed to the
company. See [`business-rules.md`](business-rules.md) §7.1.

**Company currency.** The currency the company keeps its books in. Every employee-paid expense
entry is written entirely in it. See also **receipt currency**.

**Company-paid.** The payment mode in which the company itself paid the third party, so nothing is
owed to the employee. Stored value `company_account`, labelled *Company*. Posting such an expense
creates a miscellaneous entry and a payment, and the expense goes straight to *Paid*.

**Compatibility finding.** A marker used throughout this folder for an observed behaviour that
looks like a defect. The behaviour is specified as observed, because a rebuild that departs from it
would be incompatible, and a note says what a corrected behaviour would be.

**Compounded tax.** A tax flagged as including the base amount of the taxes that follow it, so that
those later taxes are computed on a raised base. The price-included divisor becomes a product
instead of a sum.

**Content fingerprint.** The checksum computed over an attachment's bytes, used to decide whether
two expenses carry the **same receipt**. See [`calculations.md`](calculations.md) §7.2.

**Conversion rate.** The number of units of company currency obtained for one unit of receipt
currency, held on the expense to nine decimal places. Refreshed from the currency service when the
currency, the receipt total or the date changes; otherwise derived as
`total in company currency ÷ total in receipt currency`, which is what preserves a manual override.
Storage name `currency_rate`.

**Counterpart line.** The line of an entry that balances the expense and tax lines: the **payment
term line** on the employee path, the **outstanding line** on the company path.

---

## D

**Dashboard band.** The three-figure strip above the expense list and card view showing *To
Submit*, *Waiting Approval* and *Waiting Reimbursement* for the acting user, each a sum of
company-currency totals. Clicking a figure re-filters the list beneath it.

**Delivered quantity.** The quantity of a sales order line that has actually been supplied. A
rebilling line takes it from the analytic lines pointing at it, summing their unit amounts over
lines whose amount is at most zero. This is why resetting an expense's entry drives the delivered
quantity back to zero.

**Delta distribution.** The step that reconciles the sum of the individually rounded tax amounts
with the exact total tax. The difference is applied one rounding step at a time to the tax amounts
in decreasing order of absolute value. See [`calculations.md`](calculations.md) §4.6.

**Department.** The organisational unit of an employee, copied onto the expense at creation. It
carries a count of the expenses of its members awaiting approval and two window actions of its own.

**Designated expense approver.** The user named on an employee record as responsible for approving
that employee's expenses. Storage name `expense_manager_id`, labelled *Expense Approver*. Empty
means the approval falls to an Administrator or an All Approver.

**Destination account.** The account the counterpart line of a **company-paid** entry stands on:
the dedicated outstanding account of the payment method line when it has one, otherwise the
company's outstanding-payments account for outbound payments, created on the spot if the chart has
none. All expenses of one posting must resolve to the same destination account.

**Distance claim.** The canonical quantity-driven expense: a category with a non-zero unit cost
denominated per kilometre, where the employee types only the distance. See
[`calculations.md`](calculations.md) §2.4.

**Draft.** The status of an expense that has never been submitted, or that has been reset. The only
status in which the employee may freely edit and delete it, in which attachments may be both added
and removed, and in which a total of zero is tolerated. Stored value `draft`.

**Duplicate key.** The six columns on which two expenses are considered duplicates of one another:
employee, category, expense date, total in receipt currency, company and receipt currency. The
check ignores the status entirely.

**Duplicate Expense Confirmation Dialogue.** The short-lived record that lists the expenses that
look like duplicates and offers to approve them all or refuse them all. Transport name
`hr.expense.approve.duplicate`.

---

## E

**Editability.** The derived flag that says whether the acting user may change an expense's
consequential fields. It is true for the employee in *Draft*, for the expense's manager up to
*Submitted*, for an approver up to *Approved*, and for an Administrator up to *Approved*; false for
everyone once an entry exists. See [`business-rules.md`](business-rules.md) §3.2.

**Elevated rights.** A write performed with the access checks of the acting user suspended, used
where the domain must change a record the user legitimately may not edit: filling in the manager at
submission, clearing the entry reference at reset, zeroing a rebilling line on a sales order, and
storing an employee's own receipt on an expense they may no longer write.

**Embedded action.** A view offered from inside another application's screen. The project
application offers an *Expenses* embedded action in two places, both restricted to All Approvers.

**Employee.** The person who incurred the cost. Required on every expense, indexed and tracked.
Owned by [`../human-resources-core/`](../human-resources-core/); this domain adds the designated
expense approver and the search filter that limits whose expenses a user may encode.

**Employee-paid.** The payment mode in which the employee advanced the money and must be
reimbursed. Stored value `own_account`, labelled *Employee (to reimburse)*. It is the default.
Posting such an expense creates a purchase receipt with a payable term line.

**Expense.** One cost, incurred by one employee, on one date, for one expense category, in one
currency. The only durable entity the domain owns. Transport name `hr.expense`, storage name
`hr_expense`.

**Expense account.** The profit-and-loss account the cost is charged to. Resolved from the
expense's own account, else the category's, else the company's default, else the payment journal's
default; a failure to resolve any of these blocks posting.

**Expense category.** A Product Variant whose "can be expensed" flag is set, used to classify an
expense and to supply its description, unit, supplier taxes, expense account, unit cost and
rebilling policy. Six are shipped. The word *category* in this folder always means this record and
never a product category in the merchandising sense.

**Expense mailbox.** See **alias**.

**Expense Posting Dialogue.** The short-lived record that asks for the accounting date and the
journal before **employee-paid** expenses are posted. Transport name `hr.expense.post.wizard`.
Company-paid expenses never open it.

**Expense Refusal Dialogue.** The short-lived record that asks for the mandatory refusal reason.
Transport name `hr.expense.refuse.wizard`.

**Expense Split Dialogue.** The short-lived container that holds the proposed pieces of a split and
checks that they still add up to the original. Transport name `hr.expense.split.wizard`.

**Expense Split Line.** One proposed piece of an expense being split. Transport name
`hr.expense.split`.

---

## F

**Fiscal position.** The mapping that substitutes taxes and accounts according to the customer's
tax situation. Applied to the customer taxes of a rebilling line.

**Forced price-included mode.** The instruction given to the tax engine for every expense
computation: treat the figure supplied as already containing the tax. It is what makes a
price-excluded tax behave as a price-included one on an expense.

---

## G

**Grouping key (accounting).** The tuple the tax engine aggregates tax amounts by. For an expense
receipt it holds the partner, the currency, the analytic distribution, the account, the tax set,
the tax distribution line, the tax group, the tax report tags **and the expense**. Because the
expense is part of it, two expenses on one receipt bearing the same tax produce two tax lines.

**Guard.** A condition that must hold for a transition to run. Every guard of this domain, with the
exact text shown when it fails, is gathered in [`state-machines.md`](state-machines.md) §7.

---

## I

**In Payment.** The status of an employee-paid expense whose entry has been paid but whose bank
side is not yet reconciled. Stored value `in_payment`. It appears only where the full accounting
capability is active; with the invoicing capability alone the in-payment hook collapses it into
*Paid*.

**In-payment hook.** The single point at which the accounting core answers "what status does a
document take when it has been paid but the bank has not confirmed?". A rebuild must expose the
same hook and consult it in exactly this one place.

**Industry-standard default.** A marker used where the source behaviour leaves a question genuinely
open. The specification states the resolution a rebuild must implement rather than leaving the
question open.

**Internal reference.** The short code on a Product Variant, reproduced in code font. The mailbox
parser matches the first word of a subject against it, and the bulk upload operation looks for
`EXP_GEN`.

---

## J

**Journal.** The book an entry is written into. An employee-paid expense uses the purchase journal
chosen in the posting dialogue; a company-paid expense uses the journal of its payment method line,
which is a bank, cash or credit-card journal. The check that a document kind must match the journal
kind is **suspended** for entries carrying expenses.

**Journal Entry.** The accounting document produced by posting. Transport name `account.move`.
Extended here with the set of expenses it carries, with a count of them, and with expense-specific
overrides of the commercial partner, the term lines, the journal-kind check, cancellation and
reversal.

**Journal Item.** One line of a journal entry. Transport name `account.move.line`. Extended here
with a reference to the expense that produced it, and with expense-specific overrides of the
partner, the displayed totals and the payable-or-receivable check.

---

## L

**Line label.** The text on a journal item produced by an expense:
*"«employee name»: «first line of the expense description, truncated to 64 characters»"*. The same
label is reused on the outstanding line of a company-paid entry and on the rebilling line.

---

## M

**Main attachment.** The attachment shown beside the form. The **last** file uploaded through the
receipt operation is forced into this role; on a journal entry the **first** copied attachment is.

**Manager (on an expense).** The user field that names who is expected to approve this particular
expense. Filled at submission from the responsible approver when empty, and **overwritten with the
acting user** at approval. Storage name `manager_id`. It is not the employee's line manager, though
it often holds the same person.

**Mandatory plan.** An analytic plan declared compulsory for the business domain `expense`. The
approval operation validates the distribution against every such plan.

**Message subtype.** The classification broadcast on the thread when the status changes. Six are
shipped for the Expense, none subscribed to by default. See
[`state-machines.md`](state-machines.md) §3.6.

**Miscellaneous entry.** The document kind of a **company-paid** expense entry: not an invoice, so
the dynamic term-line mechanism never runs on it and its counterpart line is written explicitly.
Stored value `entry`.

---

## N

**Numbering series.** A shipped series named *Expense invoice*, code `hr.expense.invoice`, prefix
`EXP/`, three-digit padding. The Expense itself is **not** numbered by it; entries take their
numbers from their journal's own series.

---

## O

**Outstanding account.** A reconcilable account that holds money in transit — paid but not yet seen
on a bank statement. The counterpart line of a company-paid expense entry stands on one, which is
why the payable-or-receivable check is suspended for that line. See also **destination account**.

**Outstanding amount.** The part of an entry still unsettled, read from the entry onto the expense.
A *partial* payment status with a non-zero outstanding amount is reported as *In Payment*, not as
*Posted*.

---

## P

**Paid.** The status in which nothing further is owed. Reached by settling an employee-paid entry,
or immediately at posting for a company-paid expense. Stored value `paid`.

**Payable account.** The account the employee is owed on. Taken from the work contact's payable
property, falling back to its parent's, by the ordinary rule of
[`../accounts-payable/`](../accounts-payable/).

**Payment.** The record created **as the document** on the company-paid path — not as a settlement
of one — and also the record created to reimburse an employee. Transport name `account.payment`. A
payment linked to an expense is frozen against most edits.

**Payment method line.** The concrete way money moves on a journal: a manual transfer, a credit
transfer, a cheque. Chosen on a company-paid expense; its journal becomes the entry's journal and
its dedicated outstanding account, when it has one, becomes the destination account.

**Payment mode.** The field that decides which of the two accounting paths runs. Two values:
`own_account` (**employee-paid**) and `company_account` (**company-paid**). Required, tracked, and
the single most consequential field on the record.

**Payment status.** The settlement state of a journal entry: not paid, in payment, partial, paid,
reversed or blocked. Read by the expense's status computation.

**Payment term line.** The counterpart line of an employee-paid receipt, standing on the payable
account, credited with the whole receipt total, and reconcilable. One line per distinct maturity
date produced by the work contact's supplier payment term.

**Posted.** The status of an employee-paid expense that has an entry not yet settled: the period in
which the company owes the employee. Stored value `posted`. A company-paid expense never shows it.

**Price-included tax.** A tax carved **out of** a total rather than added to it. Every tax on an
expense behaves this way whatever the tax itself declares. The rule is enforced at four separate
points; see [`calculations.md`](calculations.md) §4.1.

**Priced category.** Shorthand for an expense category with a non-zero unit cost, which switches the
expense to quantity-driven pricing and forces the company currency. Its opposite is an **unpriced
category**.

**Primary bank account.** The employee's first bank account, proposed as the recipient of a
reimbursement payment and stamped onto the receipt at posting.

**Product Variant.** The catalogue record that serves as an expense category. Owned by
[`../products-and-catalog/`](../products-and-catalog/).

**Profitability panel.** The project screen that sets costs against revenues. This domain
contributes an expense section, identified `expenses`, with sequence 13, and removes four possible
double counts.

**Purchase receipt.** The document kind of an **employee-paid** expense entry: a payable document
without a vendor bill number. Stored value `in_receipt`.

---

## Q

**Quantity-driven pricing.** The pricing model used when the expense category **has** a unit cost:
the employee types only the quantity, the unit price comes from the category, the total is derived,
and the currency is forced to the company currency.

**Quiet price.** In the unit-cost change warning, one of the distinct unit prices held by the draft
expenses of the categories being edited. The warning is shown unless every draft expense would keep
exactly the amount it already has.

---

## R

**Rate label.** The human-readable rendering of the conversion rate, built as
*"1 «receipt currency name» = «rate to six decimal places» «company currency name»"* and shown only
in the multi-currency case.

**Rebilling.** Charging a customer for a cost an employee incurred, by adding a line to a sales
order when the expense's entry is posted. It produces no journal entry of its own; the entry comes
later, from the customer invoice.

**Rebilling line.** The sales order line a rebilled expense creates. It is always a **new** line:
rebilled expenses are never merged onto one line. Resetting the expense or its entry drives the
line's quantities to zero and unlinks it, but never deletes it.

**Rebilling policy.** The setting on an expense category that decides whether and how its expenses
are rebilled: *no rebilling*, *at cost* or *at sales price*.

**Receipt.** Two distinct meanings in this folder, always disambiguated by context: the **evidence**
an employee attaches to an expense, and the **purchase receipt** an employee-paid expense posts as.

**Receipt currency.** The currency of the cost as incurred. Held on the expense together with the
company-currency figures. On the employee path it never reaches the ledger; on the company path it
is the entry's own currency.

**Record rule.** A filter that decides which records a group may read or write. Twelve are shipped
by this domain, five of them on the Expense and the split dialogue and two on the accounting
entities. See [`configuration.md`](configuration.md) §8.

**Reference unit.** The unit of measure declared on an expense category, copied onto the expense
and onto the journal item, and used to convert the category's unit cost into the expense's unit.

**Refused.** The terminal status an approver puts an expense into, always with a reason, always
recoverable by a reset. Stored value `refused`. It is declared **last** in the value list so that
status-ordered views place it after every other status.

**Reimbursement.** The outgoing payment that settles an employee-paid receipt. An ordinary payment
of [`../payments-and-bank-reconciliation/`](../payments-and-bank-reconciliation/), changed here in
exactly two ways: the employee's bank account joins the batch grouping key, and the expense is
stamped onto the payment's lines.

**Reset to draft.** The operation that unwinds an expense: it reverses a posted entry in
cancellation mode, deletes a draft one, clears the approval state, the approval date and the entry
reference, and removes the approval activity.

**Responsible approver.** The user the submission operation computes when the expense names no
manager: the employee's designated expense approver, else the user of the employee's department
manager, else nothing. See [`calculations.md`](calculations.md) §3.1.

**Reversal in cancellation mode.** A reversing entry that is immediately reconciled against the
original on every reconcilable account, so the two neutralise each other. Used when a posted
expense is reset. The link between the original entry and its expenses is cleared **before** the
reversal runs, so neither entry points at the expenses afterwards.

---

## S

**Same-receipt set.** The other expenses that own an attachment whose content fingerprint matches
one of this expense's. Never populated for an expense that came out of a split, which makes the
relation asymmetric by design.

**Sales order.** The customer document a rebilled expense is added to. It must be confirmed, not
cancelled and not locked. The selector on the expense form is deliberately widened so that an
approver who cannot otherwise read sales orders can still choose one by name.

**Split.** The operation that divides one expense into several, each carrying its own total,
category, taxes and analytic distribution, with the sum constrained to equal the original. The
first piece is written onto the original record; the rest are copies of it.

**Split origin.** The reference every member of a split family carries, pointing at the root
expense. Splitting a piece again keeps the whole family pointing at the same root.

**Status.** In this folder, unqualified, the **visible** status of an expense: the derived field
whose seven values are `draft`, `submitted`, `approved`, `posted`, `in_payment`, `paid` and
`refused`. Nothing writes it directly. See [`state-machines.md`](state-machines.md) §2.

**Submitted.** The status of an expense handed to an approver. Stored value `submitted`. An
approval activity exists on the approver while it lasts.

**Subordinate employee.** An employee at or below the acting user's own employee record in the
hierarchy. Being a subordinate's approver is one of the four ways a Team Approver may act on an
expense.

**Supplier tax.** A purchase tax declared on an expense category and copied onto the expense when
the category is chosen. A category created through the expense-category action starts with **no**
supplier taxes, deliberately.

---

## T

**Tax amount.** The tax part of an expense, held twice: once in the receipt currency and once in
the company currency, each computed independently from its own total. The two are not required to
be exact multiples of one another.

**Tax distribution line.** The rule inside a tax that says what share of the tax goes to which
account and with which report tags. One tax line per distribution line is produced.

**Tax set.** The taxes named on an expense. Computed from the category and the company, writable
afterwards, restricted to purchase taxes of the expense's company.

**Team Approver.** The lowest of the three expense security levels. Sees and approves the expenses
of the employees the acting user is responsible for: their own, their department's, their
subordinates', those naming them as designated approver, and those naming them as manager. Stored
group identifier `group_hr_expense_team_approver`.

**Total in company currency.** The expense's total expressed in the company's own money. Writable,
which is how a conversion rate is overridden. Storage name `total_amount`.

**Total in receipt currency.** The expense's total as it appears on the receipt. Writable for an
unpriced category, derived for a priced one. Storage name `total_amount_currency`.

---

## U

**Unit price.** The price of one unit, always expressed in the company currency, computed and
stored **only while the expense is in *Draft***. Taken from the category when the category has a
unit cost; derived as the company-currency total divided by the quantity otherwise.

**Unit of measure.** The unit an expense is counted in — units, kilometres, days. Taken from the
category's reference unit and carried onto the journal item, which is what lets an analytic line
record a distance and a rebilling line be priced per unit.

**Unpriced category.** An expense category whose unit cost is zero, which puts the expense on the
amount-driven pricing model. See **priced category**.

**Untaxed amount.** The total minus the tax part, held twice, once per currency. It is the figure
debited to the expense account, and the figure a rebilling *at cost* recovers.

---

## V

**Vendor.** The third party the company paid on a **company-paid** expense. It becomes the entry's
partner and the payment's partner. It is required when the payment method is a credit transfer,
because the transfer file needs a creditor name. It plays no part at all on the employee path.

**Visible status.** See **status**.

---

## W

**Weekly reminder.** The only electronic mail the approval chain sends: a scheduled job running
every week that mails each approver holding submitted expenses one message with the subject
*"New expenses waiting for your approval"*.

**Work contact.** The contact record standing for the employee in the accounting system. It is the
partner of an employee-paid receipt, the holder of the payable account and of the supplier payment
term, and an employee-paid expense cannot be posted without one.

---

## Terms deliberately not used in this folder

| Not used | Because |
|---|---|
| *Expense report* | This domain has no report entity; an expense is posted individually and only the **journal entry** groups several of them. The whole-number field `former_sheet_id`, labelled *Former Report*, lets expenses that were submitted together stay groupable by a common number; it references nothing and is never validated. |
| *Expense sheet* | Same reason. |
| *Approval workflow* | There is no configurable chain: there is one approval step, whose actor is decided by the four-branch permission test. |
| *Advance* or *cash advance* | The domain has no advance entity. An employee-paid expense is an advance in substance, and the payable line is how it is tracked. |
| *Mileage rate* | The allowance is simply the **unit cost** of a category whose reference unit is *Kilometres*; nothing separate holds a rate. |
