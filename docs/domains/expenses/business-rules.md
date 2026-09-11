# Expenses — Business Rules

Every validation, constraint, invariant, permission check and locking rule of the domain, each
with a stable identifier, the condition that makes it fail, and the exact text the system shows
when it does.

**Reproduced text.** Error messages, button labels and status labels are reproduced exactly, in
quotation marks, because a rebuilt implementation must produce the same text. A few of them
contain the vendor's own name or an abbreviation; those strings are reproduced unchanged and are
the only place in this folder where such a string appears. Placeholders are written between
guillemets and described in words.

**Identifier scheme.** `EXP-«area»-«number»`, where the area is one of `INV` (invariant), `STR`
(structure), `PRM` (permission), `LIF` (life cycle), `AMT` (amount), `ACC` (account), `PST`
(posting), `PAY` (payment), `ATT` (attachment), `REB` (rebilling), `ANA` (analytic), `MAI`
(mailbox) and `EXT` (a rule this domain changes in another domain). Identifiers are stable within
this file; the index is in §13.

---

## 1. Invariants

These statements hold at every commit boundary. A rebuild that breaks one of them diverges even
when every individual rule below is honoured.

| # | Invariant |
|---|---|
| EXP-INV-1 | An Expense's visible status is **never** written directly. It is derived from the approval state and from the linked journal entry, and only from those. A rebuild that stores a status a user can set will diverge the first time an accountant deletes an entry. |
| EXP-INV-2 | The total in receipt currency and the total in company currency are both authoritative stored values. The unit price is derived from them, never the reverse — except for a quantity-driven expense, where the unit price comes from the category and the totals are derived from it. |
| EXP-INV-3 | Every tax on an expense is treated as price-included. The total the employee states is the total the company bears; a tax never increases it. |
| EXP-INV-4 | A quantity-driven expense — one whose category has a non-zero unit cost — is always denominated in the company currency while it is in *Draft*. |
| EXP-INV-5 | Beyond the *Draft* status, neither total may be zero. |
| EXP-INV-6 | An Expense belongs to exactly one company, fixed at creation and never writable afterwards. Every company-scoped record it refers to belongs to that company or to no company. |
| EXP-INV-7 | An expense that has a journal entry is accounted; an expense without one is not. There is no third state and no partial posting. |
| EXP-INV-8 | A company-paid expense owns its journal entry alone: no second expense may share it, and no second expense may share the payment that owns it. |
| EXP-INV-9 | An employee-paid expense's entry may be shared, but only with other expenses **of the same employee** posted in the same operation. |
| EXP-INV-10 | The journal entry produced for an employee-paid expense is always in the **company** currency; the one produced for a company-paid expense is always in the **receipt** currency. |
| EXP-INV-11 | Every rebilled expense owns its own sales order line. Two expenses never share one. |
| EXP-INV-12 | A rebilling line that loses its expense keeps its place on the order with both quantities at zero; it is never deleted. |
| EXP-INV-13 | An analytic account named by any expense's analytic distribution cannot be deleted. |
| EXP-INV-14 | The approval date and the manager record **who** approved and **when**; refusing does not clear either. Only a reset clears them. |

---

## 2. Structural rules on the Expense

### 2.1 EXP-STR-1 — an expense must have an employee

The employee is required by storage. Its default is the acting user's employee record, read in the
expense's company. When the acting user owns **no** employee record **and** does not belong to the
Team Approver group or a group that implies it, evaluating that default fails with:

> "The current user has no related employee. Please, create one."

The failure is a validation error raised while computing the default, so it is seen when the
creation form is opened, not when it is saved.

### 2.2 EXP-STR-2 — an expense must have a description

The description is required by storage. It is computed before insertion from the category's
display name whenever it is empty, so in practice the rule only bites when neither a description
nor a category is supplied.

### 2.3 EXP-STR-3 — an expense must have a quantity, a unit price and a currency

All three are required by storage. The quantity defaults to 1, the currency to the company
currency; the unit price is computed before insertion.

### 2.4 EXP-STR-4 — an expense must have a payment mode

Required by storage, defaulting to `own_account` (the employee payment mode). The entry form
offers the two values as radio buttons and makes them read-only once the status leaves *Draft*.

### 2.5 EXP-STR-5 — the category must be expensable, the account must not be a counterpart account

| Field | Restriction |
|---|---|
| Category (`product_id`) | only Product Variants whose "can be expensed" flag is set |
| Account (`account_id`) | the account type must **not** be receivable, payable, cash or credit card; and the account must belong to the expense's company or one of its parents |
| Taxes (`tax_ids`) | only taxes whose usage is `purchase` (purchase taxes), belonging to the expense's company |
| Payment method (`payment_method_line_id`) | only lines in the computed selectable list |
| Sales order (`sale_order_id`) | only orders whose status is `sale` (a confirmed sales order) — widened at selection time by rule EXP-REB-5 |
| Manager (`manager_id`) | only internal, non-shared users who are either the designated expense approver of some employee, or members of the Team Approver group |

### 2.6 EXP-STR-6 — automatic company consistency

The entity declares automatic company checking. Every company-scoped reference it holds — the
employee, the category, the account, each tax, the payment method line and the sales order — must
belong to the expense's company or to no company at all. A violation raises the platform's generic
company-mismatch error, specified in
[`../../overview/security-model.md`](../../overview/security-model.md), naming the offending field
and the two companies.

The same check applies to the Expense Split Line entity, which also declares it.

### 2.7 EXP-STR-7 — the company may not be changed

The company is read-only after creation. While it is momentarily empty during an interface edit,
every derived field stops recomputing and the editability flag (§3.2) becomes false, so no field
appears editable against an unknown chart of accounts. Restoring a company restores editability.

### 2.8 EXP-STR-8 — one expense per payment

A constraint on the journal-entry reference: the payment that owns an expense's journal entry may
not already carry another expense.

> "Only one expense can be linked to a particular payment"

### 2.9 EXP-STR-9 — one company-paid expense per journal entry

Checked from the entry side, over the set of expenses an entry carries: when any of them is
company-paid and the entry carries more than one expense, the entry is refused.

> "Each expense paid by the company must have a distinct and dedicated journal entry."

---

## 3. Permissions

### 3.1 The four levels of access

| Level | Group | Implies | What it adds |
|---|---|---|---|
| Employee | the internal-user group | — | Create, read, update and delete on the Expense entity, narrowed by the record rules of §3.6 to one's own draft expenses and to the expenses of employees one approves |
| Team Approver | `group_hr_expense_team_approver`, labelled *"Team Approver"* | the internal-user group | Read the expenses of one's team; approve, refuse and reset them; read journals, journal entries and journal items that carry expenses; full access to analytic lines |
| All Approver | `group_hr_expense_user`, labelled *"All Approver"* | Team Approver | Approve any expense of any employee of the accessible companies; see every expense through an unrestricted record rule |
| Administrator | `group_hr_expense_manager`, labelled *"Administrator"* | All Approver | Everything, including editing an expense that is not one's own in any editable status, and managing expense categories and activity types |

An accounting user holding the invoicing privilege (`account.group_account_invoice`) additionally
gets create, read and update — **not** delete — on the Expense entity, and is the only role that
may post. An accounting user holding the full accounting privilege
(`account.group_account_user`) is granted the same unrestricted record rule as an All Approver.

### 3.2 EXP-PRM-1 — the editability test

The editability flag (`is_editable`, "is editable by current user") is computed for the acting user
and is the single gate the entry form and several write rules use.

1. If the expense has **no company**, it is not editable. (An interface edit that momentarily
   empties the required company must not make fields look editable.)
2. If the status is not one of *Draft*, *Submitted*, *Approved* **and** the caller is not acting
   with elevated rights, it is not editable.
3. If the acting user is an expense Administrator, or is acting with elevated rights, it **is**
   editable — including for the user's own expense.
4. If the expense is the acting user's **own** and its status is *Draft*, it **is** editable.
5. Otherwise build the set of *managers* of this expense:
   - the expense's manager;
   - the employee's designated expense approver;
   - the user of the manager of the employee's department;
   - the acting user, if the acting user is an All Approver;
   - the acting user, if the expense's employee is strictly below one of the acting user's own
     employee records in the hierarchy.
6. If the expense is **not** the acting user's own **and** the acting user is in that set, it is
   editable.
7. Otherwise it is not.

The consequence worth stating: an employee may edit their own expense only while it is in *Draft*;
an approver may edit someone else's expense in *Draft*, *Submitted* or *Approved*; an
Administrator may edit their own expense in those three statuses too.

### 3.3 EXP-PRM-2 — the approval test and its four refusal reasons

The approvability flag (`can_approve`, "can approve") is true exactly when the *cannot approve*
reason for that expense is empty. The reason is computed for the acting user as follows.

Precomputed for the whole set being judged:

- *is team approver* — the acting user is a Team Approver, or is acting with elevated rights;
- *is approver* — the acting user is an All Approver, or is acting with elevated rights;
- *is administrator* — the acting user is an expense Administrator, or is acting with elevated
  rights;
- *valid companies* — the companies the acting user is currently acting for;
- *subordinate employees* — the employees of the judged expenses that are strictly below one of the
  acting user's own employee records in the hierarchy.

For each expense, the first branch that yields a reason wins:

| # | Condition | Reason text |
|---|---|---|
| 1 | the expense's company is not among the valid companies | "«expense description»: Your are neither a Manager nor a HR Officer of this expense's company" |
| 2 | the acting user is not a team approver for this expense — that is, not a Team Approver at all, the employee is not among the subordinate employees, and the acting user is not the employee's designated expense approver | "«expense description»: You are neither a Manager nor a HR Officer" |
| 3 | the acting user is not an Administrator **and** the expense's employee is the acting user's own | "«expense description»: It is your own expense" |
| 4 | the acting user is not an Administrator, is not among this expense's current managers — the employee's designated expense approver, the user of the employee's department manager, the expense's manager, plus the acting user when the employee is a subordinate — and is not an All Approver | "«expense description»: It is not from your department" |

Reason 1 reproduces a grammatical error in the shipped string: it reads "Your are" where "You are"
is meant. This is a **compatibility finding**; the string is reproduced unchanged because
automated tests and support procedures key on it. A corrected behaviour would read "You are
neither a Manager nor a HR Officer of this expense's company".

When any selected expense has a reason, the approval operation aborts with the reasons joined by
line breaks under a heading:

> "You cannot approve:"

followed by a space and then one reason per line.

### 3.4 EXP-PRM-3 — the refusal test

Identical to §3.3 — refusal uses the **same** approvability flag and the same reasons — but the
heading differs:

> "You cannot refuse:"

One difference in the assembly: the approval heading joins only the **non-empty** reasons, while
the refusal heading joins **every** reason including the empty ones. When some of the selected
expenses are refusable and others are not, the refusal message therefore contains blank lines. This
is a **compatibility finding**; a corrected behaviour would filter the empty reasons as the
approval path does.

### 3.5 EXP-PRM-4 — the reset test

The reset flag (`can_reset`, "can reset") is computed for the acting user:

```formula
can_reset = the expense's company is among the companies the user is acting for
            AND (   the user is an All Approver or an Administrator or acting with elevated rights
                 OR the expense's employee is strictly below one of the user's employees
                 OR the user is the employee's designated expense approver
                 OR ( the status is `draft` or `submitted` AND the employee is the user's own ) )
```

When any selected expense fails it, the reset operation aborts with:

> "Only HR Officers, accountants, or the concerned employee can reset to draft."

The last clause is what lets an employee pull back their own expense after submitting it, and what
stops them pulling back one that has already been approved.

### 3.6 EXP-PRM-5 — the record rules

Five rules govern who sees which Expense. A user sees a record when **at least one** of the rules
attached to a group they hold allows it, and when the global rule allows it.

| Rule | Group | Effect |
|---|---|---|
| *Manager Expense* | the full-accounting privilege, and All Approver | Allows every expense — an unconditional rule. |
| *Team Approver Expense* | Team Approver | Allows an expense whose employee is the user's own, **or** whose employee's department is managed by the user, **or** whose employee is at or below one of the user's employees, **or** whose employee's designated expense approver is the user, **or** whose manager is the user. |
| *Employee Expense* | every internal user | Allows an expense whose employee's designated expense approver is the user and whose status is *Draft*, *Submitted*, *Approved* or *Refused*; **or** whose employee is the user's own and whose status is *Draft*. |
| *Employees can't modify an expense that is not in draft state* | every internal user | A **read-only** rule (create, update and delete are switched off on it): allows reading an expense whose employee is the user's own and whose status is **not** *Draft*; or whose employee's designated expense approver is the user and whose status is *Submitted*, *Approved* or *Refused*. |
| *Expense multi company rule* | global — applies to everyone | Restricts every operation to expenses whose company is among the companies the user is acting for. |

The pair of internal-user rules is the mechanism that lets an employee **see** their submitted
expense but not change it: the permissive rule covers only the draft status, and the second rule
covers the later statuses for reading alone.

Four further rules govern the Expense Split Dialogue and are listed in
[`configuration.md`](configuration.md) §8.

Two rules extend the accounting entities for approvers: a Team Approver may read journal entries
that carry at least one expense, and journal items that carry an expense. Without them an approver
could not open the entry produced from an expense they approved.

### 3.7 EXP-PRM-6 — who may submit

Submission is refused when the acting user is **not** the expense's employee **and** the
approvability flag of §3.3 is false:

> "You do not have the required permission to submit this expense."

So an employee may always submit their own expense; anybody else may submit it only if they could
also approve it.

### 3.8 EXP-PRM-7 — the security fields may not be written

Writing any of the fields named in the guard raises:

> "You cannot edit the security fields of an expense manually"

The guarded names are the editability flag, the approvability flag and a third name, `can_refuse`,
which is **not a field of the entity**; the reset flag `can_reset`, which is a field, is **not**
guarded. This is a **compatibility finding**: the guard protects a name that does not exist and
leaves one that does unprotected. In practice the reset flag is still unwritable, because it is a
non-stored computed field with no write-back rule, so the storage layer refuses the write with its
own generic error. A corrected behaviour would guard the editability flag, the approvability flag
and the reset flag.

### 3.9 EXP-PRM-8 — writing a consequential field requires editability

Writing any of the tax set, the analytic distribution, the account or the manager onto an expense
that is **not** editable for the acting user, and without elevated rights, raises:

> "Uh-oh! You can’t edit this expense."
>
> "Reach out to the administrators, flash your best smile, and see if they'll grant you the magical access you seek."

The two lines are one message separated by a blank line. The apostrophe in "can’t" is the typographic
form, not the straight one. The same message is raised by the interface rule that fires when the
receipt-currency total is edited on the form by a user for whom the expense is not editable.

### 3.10 EXP-PRM-9 — approving through a direct write is re-checked

Writing a status of `approved` or an approval state of `approved` re-runs the approval test of
§3.3 — but only over the expenses that were **not** automatically validated, that is, those whose
manager differs from the employee's own user **or** whose employee has a designated expense
approver. Writing a status or approval state of `refused` re-runs the refusal test of §3.4 over the
whole set.

This is what stops an employee from writing an approved status directly onto their own submitted
expense, while still letting the automatic-validation path write it.

---

## 4. Life-cycle rules

### 4.1 EXP-LIF-1 — deletion is refused beyond approval

Deleting an expense whose status is *Approved*, *Posted*, *In Payment* or *Paid* raises:

> "You cannot delete a posted or approved expense."

*Draft*, *Submitted* and *Refused* expenses may be deleted, subject to the record rules. Deleting
an expense deletes its attachments with it.

### 4.2 EXP-LIF-2 — splitting is refused beyond posting

Opening the split dialogue on an expense whose status is *Posted*, *Paid* or *In Payment* raises:

> "You cannot split an expense that is already posted."

### 4.3 EXP-LIF-3 — splitting requires editability

Opening the split dialogue on an expense that is not editable for the acting user (§3.2) raises:

> "You do not have the rights to edit this expense."

### 4.4 EXP-LIF-4 — the split button is offered on different statuses to different roles

The entry form offers two split buttons, and each is visible to a different audience:

| Audience | Visible when |
|---|---|
| A user who is neither a Team Approver nor an accounting user | the status is *Draft* **and** the category has no unit cost |
| A Team Approver or an accounting user | the status is **not** *In Payment*, *Paid*, *Refused* or *Posted* — that is, *Draft*, *Submitted* or *Approved* — **and** the category has no unit cost |

A quantity-driven expense is never splittable from the form, because its amount is a function of
its quantity and splitting the amount would contradict the category's unit cost.

### 4.5 EXP-LIF-5 — refusal is refused when a linked entry is posted

Refusing an expense whose linked journal entry is in any status other than draft raises:

> "You cannot cancel an expense linked to a posted journal entry"

Draft linked entries are deleted as part of the refusal, so that no orphan entry is left behind.

### 4.6 EXP-LIF-6 — reset is refused when a linked entry is not draft

The reset operation checks, after the permission test of §3.5, that no selected expense is linked
to an entry whose status is anything other than absent or draft:

> "You cannot reset to draft an expense linked to a posted journal entry."

The guard and the operation then disagree, deliberately: the operation goes on to **reverse** every
non-draft entry. The guard is evaluated first over the whole selection, so the reversal branch is
reachable only when an entry becomes non-draft between the guard and the reversal, or when the
operation is invoked through the internal path that skips the guard. A rebuild must keep both: the
guard, because it is what a user sees, and the reversal, because it is what makes a reset safe.

### 4.7 EXP-LIF-7 — the status may not be written by an employee

An internal user without approval rights may write a status of `draft` onto their own expense — the
read-only record rule permits it and the status is recomputed anyway — but writing `approved`
raises the approval test of §3.10.

---

## 5. Posting rules

### 5.1 EXP-PST-1 — only approved expenses may be posted

> "You can only generate an accounting entry for approved expense(s)."

Checked over the whole selection before anything is created.

### 5.2 EXP-PST-2 — every expense must have a payment mode

> "Please specify if the expenses were paid by the company, or the employee."

Because the payment mode is required by storage, this is reachable only for a record whose mode was
cleared through an unusual path.

### 5.3 EXP-PST-3 — employee-paid expenses of several companies may not be posted together

> "You can't post simultaneously employee-paid expenses belonging to different companies"

Company-paid expenses are exempt: each of them produces its own entry in its own company, so a
mixed-company selection of company-paid expenses is accepted.

### 5.4 EXP-PST-4 — a credit-transfer payment method needs a vendor

For a company-paid expense whose payment method line's code is `sepa_ct` — the single-area
credit-transfer method — a vendor is required, because the transfer file needs a creditor name:

> "The vendor is required for expenses using SEPA Credit Transfer as the payment method."
>
> "Please set a vendor on the following expenses: «the descriptions of the offending expenses, separated by a comma and a space»"

The two lines are one message separated by a line break.

Conversely, a payment created from an expense is **exempt** from the requirement that the recipient
bank account be present and trusted: that check is switched off for expense payments, so a
credit-transfer expense posts even when no recipient bank account has been recorded.

### 5.5 EXP-PST-5 — a company-paid expense needs a payment method line

When the expense's payment method line is empty:

> "You need to add a manual payment method on the journal («name of the journal»)"

The placeholder renders the journal of the payment method line — which, the line being empty, is
itself empty, so the message reads with an empty pair of brackets. This is a **compatibility
finding**; a corrected behaviour would name the company rather than an empty journal.

### 5.6 EXP-PST-6 — posting requires the right to create journal entries

The posting dialogue checks that the acting user may create journal entries:

> "You don't have the rights to create accounting entries."

### 5.7 EXP-PST-7 — the posting dialogue is for employee-paid expenses only

Invoking the dialogue over a selection that contains a company-paid expense raises:

> "Only expense paid by the employee can be posted with the wizard"

The ordinary posting operation never does this: it splits the selection by payment mode and only
hands the employee-paid part to the dialogue.

### 5.8 EXP-ACC-1 — an expense account must be resolvable

When the resolution ladder of [`calculations.md`](calculations.md) §6.2 finds nothing, posting
aborts with a three-line message:

> "Odoo had a look at your expense, its product, your company and the journal but came back with empty hands."
>
> "Give Odoo a hand to find an account by setting up an expense account."
>
> "«the expense record» «the expense description»."

The first two lines name the vendor of the system being specified. They are reproduced verbatim
because support procedures and automated tests key on them; the specification's own prose never
uses that name. The third line's first placeholder renders the record itself, which displays as the
expense's description, so the line repeats the description twice. This is a **compatibility
finding**; a corrected behaviour would render an identifier and a description.

### 5.9 EXP-ACC-2 — an employee-paid expense needs a work contact

> "No work contact found for the employee «employee name», please configure one."

Raised while resolving the destination account for an employee-paid expense, because the payable
account is a property of the employee's work contact.

### 5.10 EXP-ACC-3 — one destination account per posting

> "The following expenses payment method leads to several accounts payable and this isn't supported:"
>
> "«the offending records»"

Raised when the destination-account resolution over a set of expenses yields more than one account.
The placeholder is filled from the **account** identifiers rather than the expense identifiers; see
the compatibility finding in [`calculations.md`](calculations.md) §6.3.

### 5.11 EXP-ACC-4 — the outstanding account must be active

> "The account «account name» («account code») is archived. Activate it to continue"

Offered as a redirecting warning with a button labelled *"Go to Account"* that opens the archived
account.

---

## 6. Amount rules

### 6.1 EXP-AMT-1 — no zero total beyond draft

A constraint re-evaluated on every change of status, approval state, company-currency total or
receipt-currency total:

```formula
violated = ( status ≠ `draft` OR the approval state is set )
           AND ( is_zero( company currency , total_amount )
                 OR is_zero( receipt currency , total_amount_currency ) )
```

> "Only draft expenses can have a total of 0."

Both directions are covered: an expense with a zero total cannot be submitted, and an expense that
is already submitted, approved or posted cannot have either total written to zero. The way back is
to remove the accounting, reset to draft, and only then write the zeroes — and a single write that
sets a zero total **and** a non-draft status is refused, because the constraint sees the committed
values.

| Attempt | Outcome |
|---|---|
| Submit an expense whose totals are 0.00 | refused |
| Write a receipt total of 0.00 onto a submitted expense | refused |
| Write a company total of 0.00 onto an approved expense | refused |
| Write either total to 0.00 onto a posted expense | refused |
| Reset a posted expense to draft, then write both totals to 0.00 | allowed |
| Write, in one operation, a receipt total of 0.00 and a status of `submitted` | refused |
| Write, in one operation, both totals to 0.00 and a status of `draft` | allowed |

### 6.2 EXP-AMT-2 — the quantity of an unpriced category collapses to one

An interface rule: whenever the "category has a cost" flag turns **false** while the status is
*Draft*, the quantity is forced back to 1. It fires on the form only; a write through the storage
layer does not trigger it, which is why the cascade of
[`calculations.md`](calculations.md) §2.7 sets the quantity explicitly.

### 6.3 EXP-AMT-3 — the currency of a priced category is forced to the company currency

Whenever the "category has a cost" flag is true and the status is *Draft*, the currency is
recomputed to the company currency. The computation writes nothing when the flag is false, so an
amount-driven expense keeps whatever currency the employee chose.

### 6.4 EXP-AMT-4 — the unit price is frozen outside draft

The unit-price computation returns immediately for an expense whose status is not *Draft*. A
posted expense whose totals are somehow changed keeps the unit price that produced its journal
item.

---

## 7. Rules this domain changes in other domains

### 7.1 EXP-EXT-1 — the commercial partner of an employee expense entry

For a journal entry carrying at least one **employee-paid** expense, the commercial partner is:

```formula
commercial partner = the partner's own commercial partner ,
                     unless that is the company's own partner ,
                     in which case the partner itself
```

Without this rule, an employee whose contact record hangs under the company's own contact would
have their reimbursement booked against the company — the company would, in effect, owe itself. The
ordinary rule of [`../contacts-and-organizations/`](../contacts-and-organizations/) applies to every
other entry.

### 7.2 EXP-EXT-2 — every line of an expense entry takes the entry's partner

For every journal item of an entry that carries expenses — including the payment-term line, which
carries no expense of its own — the partner is forced to the **entry's** partner instead of being
derived from the account or the document. This keeps the employee on all lines and prevents an
unrelated bank account from being proposed when the reimbursement is registered.

### 7.3 EXP-EXT-3 — the journal-kind check is suspended

The general rule that a document of a given kind must sit in a journal of the matching kind — a
purchase document in a purchase journal, a sales document in a sales journal — is **not applied**
to entries that carry expenses. A purchase receipt produced from expenses may therefore be written
into a sales, bank, cash or miscellaneous journal. The posting dialogue offers purchase journals
only, so the exemption bites when a journal is imposed through another path.

### 7.4 EXP-EXT-4 — the payable-or-receivable check is suspended for company-paid lines

The rule that a payment-term line must sit on a payable or receivable account is **skipped** for
every journal item whose expense is company-paid, because the counterpart of that path is an
outstanding-payments account and not a payable one.

### 7.5 EXP-EXT-5 — expense items are grouped separately by the tax engine

The expense identifier is added to the grouping key the tax engine uses both for base lines and for
tax-distribution lines. Two expenses on the same entry that bear the same tax therefore produce
**two** tax lines, never one merged line. The same identifier is added to the matching used to tie
a tax line back to its base line: a tax line matches a base line only when the base line carries no
expense or the two expenses are the same.

### 7.6 EXP-EXT-6 — a tax used by an expense counts as used

A tax is marked as used, and therefore protected from silent removal, as soon as at least one
expense refers to it.

### 7.7 EXP-EXT-7 — reversing or cancelling an entry unlinks its expenses

Before an entry is reversed, and after an entry is cancelled, its link to its expenses is cleared.
Neither the original nor the reversal points at the expenses afterwards. The stated reason is that
cancelling an entry is **not** cancelling the expense: the expense falls back to *Approved* and may
be reimbursed through a new entry.

### 7.8 EXP-EXT-8 — an analytic account used by an expense may not be deleted

> "You cannot delete an analytic account that is used in an expense."

Checked by scanning the expense table for any analytic distribution naming one of the accounts
being deleted.

### 7.9 EXP-EXT-9 — an analytic applicability for expenses always shows its account prefix

An analytic plan applicability whose business domain is `expense` always displays its
account-prefix criterion, because every expense carries an expense account and the prefix is
therefore always meaningful.

### 7.10 EXP-EXT-10 — the expensable flag and the purchasable flag cannot disagree

| Direction | Rule |
|---|---|
| A product whose kind is neither a goods product nor a service, or which is not purchasable, is **forced** not expensable | the computation clears the flag |
| A product that is expensable is **forced** purchasable | the computation sets the flag |

A product that may not be sold, and a product that may not be expensed, both have their rebilling
policy forced to `no` (*No*).

### 7.11 EXP-EXT-11 — a new expense category starts with no supplier taxes

When a product form is opened with the "create an expense category" intent, the default supplier
taxes are cleared, so a new category starts untaxed even in a chart of accounts that names default
purchase taxes.

---

## 8. Payment rules

### 8.1 EXP-PAY-1 — a payment linked to an expense is frozen

Writing any of the following onto a payment that carries an expense raises
> "You cannot do this modification since the payment is linked to an expense."

| Frozen field | Frozen field |
|---|---|
| date | currency |
| amount | partner |
| payment direction | destination account |
| partner kind | recipient bank account |
| payment reference | journal |
| memo | payment method line |

Every other field stays writable — the "sent" marker in particular, so that a payment file can
still be produced and acknowledged.

### 8.2 EXP-PAY-2 — a payment created from an expense never needs a recipient bank account

The requirement flag is forced off for every payment whose entry carries expenses.

### 8.3 EXP-PAY-3 — the outstanding account of an expense payment is the expense's destination

For a payment whose expenses are company-paid, the outstanding account is the destination account
resolved by [`calculations.md`](calculations.md) §6.3 — the payment method line's dedicated account,
else the company's outbound outstanding account — instead of the journal's own default.

### 8.4 EXP-PAY-4 — the reimbursement batch key carries the employee's bank account

When a line being paid belongs to an entry carrying an **employee-paid** expense and that entry
names no recipient bank account, the batching key of the register-payment dialogue is extended with
a recipient bank account taken from:

1. the employee's primary bank account, read with elevated rights;
2. otherwise the first bank account of the line's partner.

Because the recipient bank account is part of the key, two employees are never merged into one
payment, and an employee with no bank account is never merged with one who has.

### 8.5 EXP-PAY-5 — the expense is stamped onto the payment's lines

After the payments are built, every line of a payment created from expense lines is stamped with
the **first** expense found among the lines being paid, so the payment is reachable from the expense
and the expense from the payment.

---

## 9. Attachment rules

### 9.1 EXP-ATT-1 — an attachment may be added only up to approval

Creating an attachment on an expense is refused unless, for **every** expense concerned, the status
is *Draft* or *Submitted* **and** either the acting user may write the expense or the acting user is
the expense's employee — or the caller is acting with elevated rights:

> "You can't add attachments to an expense once it has been approved."

The same condition reached through the receipt-upload operation produces a differently worded
message, reproduced here as well because it is what a user of that operation sees:

> "You can't add an attachment to an expense once it has been approved."

A user with no access at all to the expense is stopped earlier, by the upload endpoint:

> "You are not allowed to upload an attachment here."

### 9.2 EXP-ATT-2 — an attachment may be deleted only up to submission

Deleting an attachment of an expense is refused unless, for every expense concerned, the expense
still exists, its status is *Draft* or *Submitted*, and the acting user may write it — or the caller
is acting with elevated rights:

> "You can't delete attachments from an expense once it has been submitted."

The asymmetry with §9.1 is deliberate and is the heart of the receipt-integrity design: an employee
may still **add** evidence after submitting, but may no longer **remove** it.

| Status | Employee, own expense | Approver | Administrator | Elevated rights |
|---|---|---|---|---|
| *Draft* | add and delete | add and delete | add and delete | add and delete |
| *Submitted* | add only | add and delete | add and delete | add and delete |
| *Approved* and beyond | neither | neither | neither | both |

### 9.3 EXP-ATT-3 — uploading a receipt requires write access or ownership

The receipt-upload operation refuses when the acting user may not write the expense **and** is not
the expense's employee:

> "You don't have the access rights to modify this expense."

When the upload produced no usable attachment — because the creation rule of §9.1 rejected it — the
operation raises the message of §9.1 instead.

### 9.4 EXP-ATT-4 — an employee's own upload is written with elevated rights

An attachment created on an expense whose employee is the acting user, where the acting user may not
otherwise write that expense, is created with elevated rights and with only four values retained:
the file name, the raw content, the owning record and the owning entity. This is what lets an
employee attach a receipt to their own **submitted** expense, which they may no longer edit.

Every other attachment creation goes through the ordinary path with the caller's own rights.

### 9.5 EXP-ATT-5 — the last uploaded file becomes the main attachment

The receipt-upload operation forcibly sets the **last** attachment of the upload as the expense's
main attachment, overriding the ordinary rule that the main attachment is chosen once and kept.

### 9.6 EXP-ATT-6 — creating expenses from files

The bulk creation operation refuses in three ways:

| Condition | Message |
|---|---|
| no attachment identifier was supplied | "No attachment was provided" |
| any supplied attachment already belongs to a record, or belongs to an entity other than the Expense entity | "Invalid attachments!" |
| no Product Variant at all has the "can be expensed" flag set | "You need to have at least one category that can be expensed in your database to proceed!" |

---

## 10. Rebilling rules

### 10.1 EXP-REB-1 — the order must be confirmed, not cancelled and not locked

Evaluated for each expense being posted, before any sales order line is created:

| Condition | Message |
|---|---|
| the order's status is a quotation or a sent quotation | "The Sales Order «order reference» to be reinvoiced must be validated before registering expenses." |
| the order is cancelled | "The Sales Order «order reference» to be reinvoiced is cancelled. You cannot register an expense on a cancelled Sales Order." |
| the order is locked | "The Sales Order «order reference» to be reinvoiced is currently locked. You cannot register an expense on a locked Sales Order." |

Each of the last two is one message with a line break after the first sentence.

### 10.2 EXP-REB-2 — the order is cleared when the expense stops being rebillable

The sales order and the rebilling line are both emptied whenever the expense's category ceases to
have a rebilling policy of `cost` or `sales_price`. The same rule applies to an Expense Split Line.

### 10.3 EXP-REB-3 — a rebilled expense always gets its own line

The reuse of an existing sales order line — which the generic rebilling of vendor bills performs for
a line priced at sales price on a product invoiced on delivered quantities — is switched **off** for
expenses. Every rebilled expense is forced onto a new line, so that quantities can be adjusted or
reset per expense without disturbing another.

### 10.4 EXP-REB-4 — resetting unlinks the line and zeroes it

Whenever a posted expense entry is reset to draft, reversed, or deleted, every rebilling line of the
expenses it carried is written back to an ordered quantity of 0, a delivered quantity of 0, and no
linked expenses. The line stays on the order. The write is performed with elevated rights after
checking that the caller may write the **expense**, because an employee who may edit their expense
usually may not edit a sales order.

### 10.5 EXP-REB-5 — the order selector is widened for expenses

When the sales order selector is opened with the "show every order for expenses" intent, and the
acting user is a salesperson who may **not** see all leads, the ordinary name search is replaced by
an elevated search restricted to confirmed orders of the accessible companies. The effect is that an
employee may name any confirmed order as the order to rebill to, even one belonging to another
salesperson, while still seeing only its name. Negative search operators are refused under this
intent as not implemented.

### 10.6 EXP-REB-6 — changing the order re-derives the analytic distribution

Changing the sales order on the form invalidates the analytic distribution of every line whose
distribution is not protected and queues it for recomputation, so the reconciliation of
[`calculations.md`](calculations.md) §11.3 runs again.

### 10.7 EXP-REB-7 — the rebilling policy is visible to approvers

The rebilling policy of a product is normally shown only for purchasable products and only to users
who may set rebilling policies. For a product marked as an expense category it is additionally
shown whenever the acting user is at least an All Approver.

---

## 11. Analytic rules

### 11.1 EXP-ANA-1 — mandatory plans are validated at approval

The analytic distribution is validated against every analytic plan declared **mandatory** for the
business domain `expense`, passing the expense's account, its category and its company. The
validation runs:

- inside the approval algorithm, for every expense being approved;
- therefore also on the automatic-validation path, where submission approves immediately.

A failure raises the analytic validation error of
[`../analytic-accounting/business-rules.md`](../analytic-accounting/business-rules.md), whose text
states that one or more lines require a distribution totalling one hundred per cent.

The entry form and the posting dialogue both pass the "validate analytic" intent, so the check also
runs when an approver presses *Approve* on the form and when an accountant posts.

### 11.2 EXP-ANA-2 — the distribution must total one hundred per cent per plan

The generic analytic invariant applies unchanged: within each root plan, the percentages of a
distribution must add up to exactly one hundred, at the *Percentage Analytic* precision.

### 11.3 EXP-ANA-3 — a computed distribution never clears a manual one

Every distribution computation in this domain writes only when it has something to write. A
distribution model that does not match, or a project that has no analytic account, leaves the
existing distribution untouched.

---

## 12. Mailbox rules

### 12.1 EXP-MAI-1 — an unrecognised sender does not create an expense

When the sender address matches no employee, the message is handed back to the generic routing of
[`../messaging-and-activities/`](../messaging-and-activities/) and no expense is created.

### 12.2 EXP-MAI-2 — the mailbox accepts employees only

The shipped alias has a contact policy of `employees`: only messages from an address belonging to
an internal user are routed to it. Messages from anyone else are rejected by the routing layer
before this domain sees them.

### 12.3 EXP-MAI-3 — an expense created by mail may have no category

The category is deliberately **not** required by storage, so that a message whose subject names no
recognisable category still produces an expense. Submission then refuses it:

> "You can not submit an expense without a category."

### 12.4 EXP-MAI-4 — the acknowledgement is sent to the employee, not to the sender

When the employee has a user, the acknowledgement is posted as a note on the expense's message
thread, addressed to that user's partner, with the subject *"Re: «the original subject»"*. When the
employee has **no** user, an electronic-mail message is sent directly to the address the original
message came from, with the same subject and with the original message referenced.

---

## 13. Index of rules

| Identifier | Subject | Section |
|---|---|---|
| EXP-INV-1 … EXP-INV-14 | Invariants | §1 |
| EXP-STR-1 | An expense must have an employee | §2.1 |
| EXP-STR-2 | An expense must have a description | §2.2 |
| EXP-STR-3 | Quantity, unit price and currency are required | §2.3 |
| EXP-STR-4 | A payment mode is required | §2.4 |
| EXP-STR-5 | Field-level restrictions | §2.5 |
| EXP-STR-6 | Automatic company consistency | §2.6 |
| EXP-STR-7 | The company may not be changed | §2.7 |
| EXP-STR-8 | One expense per payment | §2.8 |
| EXP-STR-9 | One company-paid expense per journal entry | §2.9 |
| EXP-PRM-1 | The editability test | §3.2 |
| EXP-PRM-2 | The approval test and its reasons | §3.3 |
| EXP-PRM-3 | The refusal test | §3.4 |
| EXP-PRM-4 | The reset test | §3.5 |
| EXP-PRM-5 | The record rules | §3.6 |
| EXP-PRM-6 | Who may submit | §3.7 |
| EXP-PRM-7 | The security fields may not be written | §3.8 |
| EXP-PRM-8 | Writing a consequential field requires editability | §3.9 |
| EXP-PRM-9 | Approving through a direct write is re-checked | §3.10 |
| EXP-LIF-1 | Deletion is refused beyond approval | §4.1 |
| EXP-LIF-2 | Splitting is refused beyond posting | §4.2 |
| EXP-LIF-3 | Splitting requires editability | §4.3 |
| EXP-LIF-4 | Who sees the split button when | §4.4 |
| EXP-LIF-5 | Refusal is refused when a linked entry is posted | §4.5 |
| EXP-LIF-6 | Reset is refused when a linked entry is not draft | §4.6 |
| EXP-LIF-7 | The status may not be written by an employee | §4.7 |
| EXP-PST-1 | Only approved expenses may be posted | §5.1 |
| EXP-PST-2 | Every expense must have a payment mode | §5.2 |
| EXP-PST-3 | One company per employee-paid posting | §5.3 |
| EXP-PST-4 | A credit-transfer method needs a vendor | §5.4 |
| EXP-PST-5 | A company-paid expense needs a payment method line | §5.5 |
| EXP-PST-6 | Posting requires the right to create entries | §5.6 |
| EXP-PST-7 | The posting dialogue is for employee-paid expenses | §5.7 |
| EXP-ACC-1 | An expense account must be resolvable | §5.8 |
| EXP-ACC-2 | An employee-paid expense needs a work contact | §5.9 |
| EXP-ACC-3 | One destination account per posting | §5.10 |
| EXP-ACC-4 | The outstanding account must be active | §5.11 |
| EXP-AMT-1 | No zero total beyond draft | §6.1 |
| EXP-AMT-2 | The quantity of an unpriced category collapses to one | §6.2 |
| EXP-AMT-3 | The currency of a priced category is the company currency | §6.3 |
| EXP-AMT-4 | The unit price is frozen outside draft | §6.4 |
| EXP-EXT-1 | The commercial partner of an employee expense entry | §7.1 |
| EXP-EXT-2 | Every line takes the entry's partner | §7.2 |
| EXP-EXT-3 | The journal-kind check is suspended | §7.3 |
| EXP-EXT-4 | The payable-or-receivable check is suspended | §7.4 |
| EXP-EXT-5 | Expense items are grouped separately by the tax engine | §7.5 |
| EXP-EXT-6 | A tax used by an expense counts as used | §7.6 |
| EXP-EXT-7 | Reversing or cancelling unlinks the expenses | §7.7 |
| EXP-EXT-8 | An analytic account in use may not be deleted | §7.8 |
| EXP-EXT-9 | An expense applicability shows its account prefix | §7.9 |
| EXP-EXT-10 | The expensable and purchasable flags agree | §7.10 |
| EXP-EXT-11 | A new expense category starts untaxed | §7.11 |
| EXP-PAY-1 | A payment linked to an expense is frozen | §8.1 |
| EXP-PAY-2 | No recipient bank account is required | §8.2 |
| EXP-PAY-3 | The outstanding account of an expense payment | §8.3 |
| EXP-PAY-4 | The reimbursement batch key | §8.4 |
| EXP-PAY-5 | The expense is stamped onto the payment's lines | §8.5 |
| EXP-ATT-1 | Attachments may be added up to approval | §9.1 |
| EXP-ATT-2 | Attachments may be deleted up to submission | §9.2 |
| EXP-ATT-3 | Uploading requires write access or ownership | §9.3 |
| EXP-ATT-4 | An employee's own upload uses elevated rights | §9.4 |
| EXP-ATT-5 | The last uploaded file becomes the main attachment | §9.5 |
| EXP-ATT-6 | Creating expenses from files | §9.6 |
| EXP-REB-1 | The order must be confirmed, open and unlocked | §10.1 |
| EXP-REB-2 | The order is cleared when rebilling stops | §10.2 |
| EXP-REB-3 | A rebilled expense always gets its own line | §10.3 |
| EXP-REB-4 | Resetting unlinks the line and zeroes it | §10.4 |
| EXP-REB-5 | The order selector is widened | §10.5 |
| EXP-REB-6 | Changing the order re-derives the distribution | §10.6 |
| EXP-REB-7 | The rebilling policy is visible to approvers | §10.7 |
| EXP-ANA-1 | Mandatory plans are validated at approval | §11.1 |
| EXP-ANA-2 | The distribution must total one hundred per cent | §11.2 |
| EXP-ANA-3 | A computed distribution never clears a manual one | §11.3 |
| EXP-MAI-1 | An unrecognised sender creates nothing | §12.1 |
| EXP-MAI-2 | The mailbox accepts employees only | §12.2 |
| EXP-MAI-3 | An expense created by mail may have no category | §12.3 |
| EXP-MAI-4 | Where the acknowledgement goes | §12.4 |
