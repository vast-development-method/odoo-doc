# Expenses

## Scope

This domain specifies everything the system does with **costs incurred by an employee on behalf of the company**: the capture of a single expense (by hand, from a photograph or scan of a receipt, by electronic mail, by splitting an existing expense, or from a project or sales context), the pricing of that expense from an expense category, the treatment of the taxes that the receipt already includes, the conversion of a foreign-currency receipt into the company currency, the approval chain that decides who may submit, approve, refuse, reset and post it, the accounting entries produced when it is posted — different for a cost the employee advanced and for a cost the company itself paid — the reimbursement of the employee and the resulting payment state, and the rebilling of the cost to a customer on a sales order.

Everything in this folder is derived from the behaviour of five capability packages:

| Capability package (as a role, not a name) | What it contributes |
|---|---|
| **Expenses** | The Expense entity, expense categories, the approval chain, the split, refusal, duplicate-approval and posting dialogues, the mailbox that turns messages into expenses, the printable expense document, the security groups and record rules, the weekly reminder job, and every journal entry the domain produces |
| **Expense rebilling on sales** | The link from an Expense to a Sales Order, the three rebilling policies on the expense category, the creation and quantity maintenance of the rebilling Sales Order Line, and the reset of that line when the entry is reversed or deleted |
| **Expense margin on sales** | The cost side of the margin of a rebilling Sales Order Line, taken from the untaxed amount of the originating expense |
| **Expense costs on projects** | The default analytic distribution taken from the project in context, the project's expense action, and the expense cost section of the project profitability panel |
| **Expense costs on projects sold** | The reconciliation of the project analytic distribution with the sales-order analytic distribution, the creation of the project's analytic account at posting time, and the revenue side of the expense section of the project profitability panel |

## What this domain does **not** re-specify

| Subject | Where it lives |
|---|---|
| The generic posting mechanics of a journal entry, its numbering, its lock dates, its hash chain, its reconciliation core | `../general-ledger/` |
| The payable side of a purchase document, the payable account selection rules, the reversal mechanics, mass posting | `../accounts-payable/` |
| Payments, the register-payment dialogue, outstanding accounts, bank statements, reconciliation models | `../payments-and-bank-reconciliation/` |
| Tax computation itself (the base, the distribution, the rounding of a tax group) | `../taxes/` |
| Currency rates, the conversion algorithm, exchange differences | `../multi-currency/` |
| Analytic plans, analytic distribution validation, analytic line creation from journal items | `../analytic-accounting/` |
| Products and product categories, units of measure | `../products-and-catalog/`, `../units-of-measure-and-packaging/` |
| Sales orders, their confirmation, their invoicing, delivered-quantity mechanics | `../sales/` |
| Employees, departments, work contacts, bank accounts of employees | `../human-resources-core/` |
| Message threads, activities, aliases, incoming mail routing | `../messaging-and-activities/` |
| Projects, project analytic accounts, the profitability panel framework | `../projects-and-tasks/` |

Where the expense domain **changes** the behaviour of one of those subjects — and it changes several — the change is stated here in full, and the unchanged part is linked. The most important such changes are:

1. Every tax on an expense behaves as a **price-included** tax, whatever the tax itself says (`calculations.md` §4).
2. A journal entry that carries expenses is exempted from the journal-versus-document-type consistency check, so an expense entry may sit in a bank or cash journal (`business-rules.md` §7.3).
3. For a company-paid expense the payable/receivable nature check on the counterpart line is suspended, because the counterpart is an outstanding-payments account, not a payable one (`business-rules.md` §7.4).
4. The payment-term line of an expense entry is rebuilt with a maturity date of today and an account chosen by the payment mode, instead of by the partner's payable property (`accounting-effects.md` §4).
5. A rebilled expense always gets **its own** Sales Order Line; rebilled expenses are never merged onto one line (`workflows.md` §9).

## Capabilities covered

| Capability | Where specified |
|---|---|
| The Expense entity with all eighty-four of its fields, its computed chain and its ordering | `entities.md` §2 |
| Expense categories: the "can be expensed" flag, the unit cost, the unit of measure, the supplier taxes, the expense account, the rebilling policy, the six shipped categories | `entities.md` §3, `configuration.md` §4 |
| The two payment modes and everything that depends on them | `entities.md` §2.7, `accounting-effects.md` §3–§5 |
| Pricing from a category that has a unit cost versus a category that has none | `calculations.md` §2 |
| The price-included tax formulas in both directions, with the base, the tax and the total | `calculations.md` §4 |
| Distance claims: quantity times unit cost, unit of measure handling | `calculations.md` §2.4, `acceptance-criteria.md` §4 |
| Foreign currency: the rate lookup, the manual rate override, the two amount pairs, the rate label | `calculations.md` §5 |
| The single state field and the two fields that drive it (approval state and the linked entry) | `state-machines.md` §2 |
| Submission, automatic validation when there is no approver, approval, refusal with a reason, reset to draft | `state-machines.md` §3, `workflows.md` §3–§6 |
| Who may submit, approve, refuse, reset, edit and post, with the exact refusal reasons | `business-rules.md` §3, `configuration.md` §6 |
| Duplicate detection on the six-column key, same-receipt detection by content checksum, and the confirmation dialogue | `calculations.md` §7, `workflows.md` §4.4 |
| Receipts: attachment rules, the main attachment, the digitisation contract, the share target | `workflows.md` §2.3, `interfaces.md` §7, `business-rules.md` §9 |
| Creation from an electronic mail message: employee identification, category parsing, amount and currency parsing, the acknowledgement message | `calculations.md` §8, `workflows.md` §2.4 |
| Splitting one expense into several, with the rounding of the halves and the transfer of attachments | `calculations.md` §9, `workflows.md` §7 |
| Posting an employee-paid expense: the grouping by employee, the journal choice, the accounting date, the entry line by line | `accounting-effects.md` §3, `workflows.md` §8 |
| Posting a company-paid expense: the payment record, the entry line by line, the outstanding account | `accounting-effects.md` §4, `workflows.md` §8.4 |
| Reimbursement, the payment state, the in-payment and paid distinction | `state-machines.md` §4, `accounting-effects.md` §6 |
| Resetting a posted expense: reversal with cancellation, deletion of a draft entry, unlinking | `workflows.md` §6, `accounting-effects.md` §7 |
| Rebilling at cost and at sales price, the line created, the quantity maintenance, the reset | `workflows.md` §9, `calculations.md` §10 |
| The margin contribution of a rebilled expense | `calculations.md` §10.5 |
| The project analytic default, the project profitability contribution | `calculations.md` §11, `workflows.md` §10 |
| The personal dashboard figures | `calculations.md` §12, `interfaces.md` §3.6 |
| Settings, the mailbox alias, the sequence, security groups, the access matrix, record rules, the weekly job | `configuration.md` |
| Every window action, view, dialogue, named operation, printable document and message template | `interfaces.md` |
| Fifty-eight numbered acceptance scenarios with concrete numbers | `acceptance-criteria.md` |

## Entities

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Expense | `hr.expense` | `hr_expense` | One cost incurred by one employee on one date for one expense category; the only durable entity the domain owns |
| Expense Split Line | `hr.expense.split` | `hr_expense_split` | One proposed piece of an expense being split; short-lived |
| Expense Split Dialogue | `hr.expense.split.wizard` | `hr_expense_split_wizard` | The container that holds the proposed pieces and checks that they still add up; short-lived |
| Expense Posting Dialogue | `hr.expense.post.wizard` | `hr_expense_post_wizard` | Asks for the accounting date and the journal before employee-paid expenses are posted; short-lived |
| Expense Refusal Dialogue | `hr.expense.refuse.wizard` | `hr_expense_refuse_wizard` | Asks for the mandatory refusal reason; short-lived |
| Duplicate Expense Confirmation Dialogue | `hr.expense.approve.duplicate` | `hr_expense_approve_duplicate` | Shows the expenses that look like duplicates and offers to approve them all or refuse them all; short-lived |
| Journal Entry | `account.move` | `account_move` | Extended with the set of expenses it carries and with expense-specific overrides of numbering, term lines, cancellation and reversal |
| Journal Item | `account.move.line` | `account_move_line` | Extended with a reference to the expense that produced it, and with expense-specific overrides of partner, tax totals and payable checks |
| Payment | `account.payment` | `account_payment` | Extended for the company-paid mode: a payment record is created **as the document**, not as a settlement of one |
| Product Variant used as expense category | `product.product` | `product_product` | Extended with the "can be expensed" flag and the rebilling policy; a category carries the unit cost, unit, taxes and expense account |
| Employee | `hr.employee` | `hr_employee` | Extended with the designated expense approver and with the search filter that limits whose expenses a user may encode |
| Department | `hr.department` | `hr_department` | Extended with the count of expenses awaiting approval |
| Company | `res.company` | `res_company` | Extended with the default expense journal and the list of payment methods allowed for company-paid expenses |
| Sales Order | `sale.order` | `sale_order` | Extended with the expenses rebilled onto it and their count |
| Sales Order Line | `sale.order.line` | `sale_order_line` | Extended with the expenses that produced it and with the single expense used for its cost |
| Analytic Plan Applicability | `account.analytic.applicability` | `account_analytic_applicability` | Extended with an `expense` business domain so a plan can be made mandatory for expenses |

## Reading order

1. **`glossary.md`** — read the twelve or so terms that carry precise meaning here (expense category, payment mode, approval state, rebilling policy, outstanding account, work contact) before anything else.
2. **`entities.md`** — the Expense entity and the four short-lived dialogue entities, field by field.
3. **`state-machines.md`** — the single visible status, the hidden approval state, and how the linked journal entry overrides both.
4. **`calculations.md`** — pricing, the price-included tax arithmetic, currency conversion, duplicate keys, mail parsing, splitting, rebilling amounts.
5. **`accounting-effects.md`** — the critical chapter: the journal entry produced by each payment mode, line by line.
6. **`workflows.md`** — the end-to-end operational sequences that tie the above together.
7. **`business-rules.md`** — every validation, every message, every permission check.
8. **`configuration.md`** — settings, groups, access rights, record rules, shipped data, scheduled job.
9. **`interfaces.md`** — navigation, views, named operations, documents, message templates.
10. **`acceptance-criteria.md`** — the numbered scenarios a rebuild must reproduce exactly.

## Dependencies on other domains

| This domain needs | For what |
|---|---|
| `../general-ledger/` | Journal entries, journals, accounts, posting, the balance invariant, reversal |
| `../accounts-payable/` | The payable term line of a purchase document, the payable account property of a partner, the reversal-with-cancellation behaviour |
| `../payments-and-bank-reconciliation/` | The Payment entity, payment method lines, outstanding accounts, the register-payment dialogue, reconciliation |
| `../taxes/` | The tax engine that turns a base line into a base amount and tax amounts |
| `../multi-currency/` | The conversion rate for a date and company, the rounding of a monetary amount |
| `../analytic-accounting/` | Analytic distribution storage, distribution models, mandatory-plan validation |
| `../products-and-catalog/` | The product variant that serves as an expense category, its accounts and its taxes |
| `../units-of-measure-and-packaging/` | The unit of an expense category and the unit-price conversion |
| `../human-resources-core/` | Employees, their work contact, their manager, their department, their bank account |
| `../messaging-and-activities/` | The message thread, the approval activity, the mailbox alias, the message templates |
| `../sales/` | Sales orders and sales order lines for rebilling; the delivered-quantity mechanism |
| `../pricing-and-pricelists/` | The sales price used by the "at sales price" rebilling policy |
| `../projects-and-tasks/` | The project analytic account and the profitability panel |

## Which domains depend on this one

| Domain | What it takes from here |
|---|---|
| `../accounts-payable/` | Expense entries are purchase receipts and appear among payable documents |
| `../payments-and-bank-reconciliation/` | The payable term line of an employee-paid entry is reconciled by a reimbursement payment; the employee's bank account is proposed by the register-payment dialogue |
| `../sales/` | Rebilling lines appear on sales orders and are invoiced by the normal invoicing flow |
| `../projects-and-tasks/` | The expense cost and revenue sections of the profitability panel |
| `../analytic-accounting/` | Analytic accounts may not be deleted while an expense still refers to them |

## Conventions used in this folder

- **Debit** means a positive balance expressed in the company currency; **credit** means a negative one. A line's *amount in currency* is stated in the document currency and always carries the same sign as its balance.
- `round_to(currency, x)` means rounding `x` to the rounding step of that currency by the half-away-from-zero method described in `../multi-currency/calculations.md`.
- A quantity written in words inside a formula (for example *total amount in currency*) names a stored field of the entity under discussion; the storage name of that field is given in the field table of `entities.md`.
- Each reproduced storage or transport name in code font is accompanied, on its first use in a document, by its full name in words.
- "Industry-standard default" marks a statement that the source behaviour leaves implicit and that a rebuild must implement in the stated way to be equivalent.
