# Expenses

## Scope

This domain specifies everything the system does with **costs incurred by an employee on behalf of the company**: the capture of a single expense (by hand, from a photograph or scan of a receipt, by electronic mail, by splitting an existing expense, or from a project or sales context), the pricing of that expense from an expense category, the treatment of the taxes that the receipt already includes, the conversion of a foreign-currency receipt into the company currency, the approval chain that decides who may submit, approve, refuse, reset and post it, the accounting entries produced when it is posted — different for a cost the employee advanced and for a cost the company itself paid — the reimbursement of the employee and the resulting payment state, and the rebilling of the cost to a customer on a sales order.

Everything in this folder is derived from the behaviour of seven capability packages:

| Capability package (as a role, not a name) | What it contributes |
|---|---|
| **Expenses** | The Expense entity, expense categories, the approval chain, the split, refusal, duplicate-approval and posting dialogues, the mailbox that turns messages into expenses, the printable expense document, the security groups and record rules, the weekly reminder job, and every journal entry the domain produces |
| **Expense rebilling on sales** | The link from an Expense to a Sales Order, the three rebilling policies on the expense category, the creation and quantity maintenance of the rebilling Sales Order Line, and the reset of that line when the entry is reversed or deleted |
| **Expense margin on sales** | The cost side of the margin of a rebilling Sales Order Line, taken from the untaxed amount of the originating expense |
| **Expense costs on projects** | The default analytic distribution taken from the project in context, the project's expense action, and the expense cost section of the project profitability panel |
| **Expense costs on projects sold** | The reconciliation of the project analytic distribution with the sales-order analytic distribution, the creation of the project's analytic account at posting time, and the revenue side of the expense section of the project profitability panel |
| **Expense document layout for one jurisdiction** | A document-title block that sets the title of the printable expense document to *"Expenses Report"* so that it conforms to a standardised business-letter layout; specified in `configuration.md` §18 and `interfaces.md` §8.4 |
| **Expense dashboard document** | A shipped spreadsheet dashboard whose main data source is the Expense entity, together with the sample version used before any expense exists; specified in `configuration.md` §17 |

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
| The Expense entity with all fifty-four of its named fields, the message-thread fields it inherits, its computed chain and its ordering | `entities.md` §2 |
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
| One hundred and thirty-one numbered acceptance scenarios with concrete numbers | `acceptance-criteria.md` |

## Entities

### Entities this domain owns

Six, of which one is durable and five are short-lived dialogue records. Each links to its generated
reference page.

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Expense | [`hr.expense`](../../references/entities/hr.expense.md) | `hr_expense` | One cost incurred by one employee on one date for one expense category; the only durable entity the domain owns |
| Expense Split Line | [`hr.expense.split`](../../references/entities/hr.expense.split.md) | `hr_expense_split` | One proposed piece of an expense being split; short-lived |
| Expense Split Dialogue | [`hr.expense.split.wizard`](../../references/entities/hr.expense.split.wizard.md) | `hr_expense_split_wizard` | The container that holds the proposed pieces and checks that they still add up; short-lived |
| Expense Posting Dialogue | [`hr.expense.post.wizard`](../../references/entities/hr.expense.post.wizard.md) | `hr_expense_post_wizard` | Asks for the accounting date and the journal before employee-paid expenses are posted; short-lived |
| Expense Refusal Dialogue | [`hr.expense.refuse.wizard`](../../references/entities/hr.expense.refuse.wizard.md) | `hr_expense_refuse_wizard` | Asks for the mandatory refusal reason; short-lived |
| Duplicate Expense Confirmation Dialogue | [`hr.expense.approve.duplicate`](../../references/entities/hr.expense.approve.duplicate.md) | `hr_expense_approve_duplicate` | Shows the expenses that look like duplicates and offers to approve them all or refuse them all; short-lived |

### Entities owned elsewhere that this domain extends

Every one of these is specified in the folder named in the last column; only the additions and the
overrides this domain makes are specified here, in `entities.md` §7 to §10.

| Entity | Transport name | Storage name | What this domain adds | Owned by |
|---|---|---|---|---|
| Journal Entry | `account.move` | `account_move` | The set of expenses it carries, their count, the exemption from the journal-kind check, the commercial-partner rule, the term-line override, and the unlinking of expenses on cancellation and reversal | [`../general-ledger/`](../general-ledger/) |
| Journal Item | `account.move.line` | `account_move_line` | A reference to the expense that produced it, and expense-specific overrides of the partner, the displayed totals, the payable check and the attachment lookup | [`../general-ledger/`](../general-ledger/) |
| Payment | `account.payment` | `account_payment` | The company-paid mode, in which a payment record is created **as the document**, not as a settlement of one; the outstanding-account override; the frozen fields | [`../payments-and-bank-reconciliation/`](../payments-and-bank-reconciliation/) |
| Payment Registration Dialogue | `account.payment.register` | `account_payment_register` | The batch key extended with the employee's bank account, and the stamping of the expense onto the payment's lines | [`../payments-and-bank-reconciliation/`](../payments-and-bank-reconciliation/) |
| Tax | `account.tax` | `account_tax` | The expenses that use the tax, the "is used" flag, and the expense carried through the grouping keys | [`../taxes/`](../taxes/) |
| Analytic Account | `account.analytic.account` | `account_analytic_account` | The deletion guard for an account named by an expense's distribution | [`../analytic-accounting/`](../analytic-accounting/) |
| Analytic Plan Applicability | `account.analytic.applicability` | `account_analytic_applicability` | The `expense` business domain, so a plan can be made mandatory for expenses, and the always-shown account prefix | [`../analytic-accounting/`](../analytic-accounting/) |
| Product Template and Product Variant used as expense category | `product.template`, `product.product` | `product_template`, `product_product` | The "can be expensed" flag, the rebilling-policy explanation, the unit-cost change warning and the cascade of a new unit cost onto draft expenses | [`../products-and-catalog/`](../products-and-catalog/) |
| Employee | `hr.employee` | `hr_employee` | The designated expense approver and the search filter that limits whose expenses a user may encode | [`../human-resources-core/`](../human-resources-core/) |
| Public Employee Profile | `hr.employee.public` | `hr_employee_public` | The same two fields, read-only | [`../human-resources-core/`](../human-resources-core/) |
| Department | `hr.department` | `hr_department` | The count of expenses awaiting approval and two window actions | [`../human-resources-core/`](../human-resources-core/) |
| Company | `res.company` | `res_company` | The default expense journal and the list of payment methods allowed for company-paid expenses | [`../contacts-and-organizations/`](../contacts-and-organizations/) |
| Settings | `res.config.settings` | `res_config_settings` | The mailbox switch, its local part and domain, the three optional capability switches and the two accounting settings | [`../platform-foundation/`](../platform-foundation/) |
| Attachment | `ir.attachment` | `ir_attachment` | The creation and deletion guards keyed on the expense status, and the elevated creation path for an employee's own receipt | [`../platform-foundation/`](../platform-foundation/) |
| Printable Document Action | `ir.actions.report` | `ir_act_report_xml` | The hook that appends every receipt of the expense to the printed expense document | [`../platform-foundation/`](../platform-foundation/) |
| Sales Order | `sale.order` | `sale_order` | The expenses rebilled onto it, their count, and the widened name search | [`../sales/`](../sales/) |
| Sales Order Line | `sale.order.line` | `sale_order_line` | The expenses that produced it, the single expense used for its cost, and the cost side of its margin | [`../sales/`](../sales/) |
| Project | `project.project` | `project_project` | The expense section of the profitability panel, the embedded expenses action, and the four exclusions that stop double counting | [`../projects-and-tasks/`](../projects-and-tasks/) |

The candidate entity list for this folder also named the generic platform entities whose transport
names begin `ir.`, `base.`, `report.`, `format.`, `properties.` and `change.`. Those belong to the
platform foundation and to [`../../overview/README.md`](../../overview/README.md); the two this
domain genuinely extends — the attachment and the printable document action — are in the table
above, and nothing else from that group is specified here.

## Reading order

1. **[`glossary.md`](glossary.md)** — read at least the terms that carry a precise and easily mistaken meaning here (expense category, payment mode, approval state, visible status, rebilling policy, destination account, outstanding account, work contact, priced and unpriced category) before anything else; the file defines every term of the domain.
2. **`entities.md`** — the Expense entity and the four short-lived dialogue entities, field by field.
3. **`state-machines.md`** — the single visible status, the hidden approval state, and how the linked journal entry overrides both.
4. **`calculations.md`** — pricing, the price-included tax arithmetic, currency conversion, duplicate keys, mail parsing, splitting, rebilling amounts.
5. **`accounting-effects.md`** — the critical chapter: the journal entry produced by each payment mode, line by line.
6. **`workflows.md`** — the end-to-end operational sequences that tie the above together.
7. **`business-rules.md`** — every validation, every message, every permission check.
8. **`configuration.md`** — settings, groups, access rights, record rules, shipped data, scheduled job.
9. **`interfaces.md`** — navigation, views, named operations, documents, message templates.
10. **`acceptance-criteria.md`** — the one hundred and thirty-one numbered scenarios a rebuild must reproduce exactly.

## Every file in this folder

| File | What it holds |
|---|---|
| [`README.md`](README.md) | This file: scope, capability packages, the entities the folder owns, the reading order, the dependencies in both directions, and the conventions used throughout |
| [`entities.md`](entities.md) | The Expense entity field by field, the four short-lived dialogue entities, the expense category, and every field this domain adds to an entity owned elsewhere; uniqueness, ordering, display, archival and company behaviour; links to the generated reference pages |
| [`state-machines.md`](state-machines.md) | The visible status and its seven values, the hidden approval state, the payment progress of the linked entry, every transition with its guards and side effects, the message subtypes broadcast, and a diagram |
| [`workflows.md`](workflows.md) | The end-to-end procedures: the six ways to capture an expense, submitting, approving, refusing, resetting, splitting, posting, rebilling, project tracking, reimbursing, and the weekly reminder |
| [`business-rules.md`](business-rules.md) | Every validation, constraint, invariant, permission check and locking rule, numbered `EXP-…`, each with its exact message, and an index of rule identifiers |
| [`calculations.md`](calculations.md) | Every formula: pricing, the price-included tax arithmetic, currency conversion and the rate override, label and account resolution, duplicate and same-receipt detection, mail parsing, splitting, rebilling prices and quantities, analytic distribution, profitability and the dashboard |
| [`accounting-effects.md`](accounting-effects.md) | The two journal entries the domain produces, item by item, with nine worked examples; reimbursement; reversal; the accounts used and the cross-checks against the payable model |
| [`configuration.md`](configuration.md) | Settings, system parameters, shipped categories, the numbering series, security groups, access rights, record rules, company configuration, the mailbox, the activity type, message subtypes and templates, the scheduled job, the digest tip, the tour, the dashboard document, the layout variant and the analytic configuration |
| [`interfaces.md`](interfaces.md) | Menus, views, window actions, named operations, dialogues, receipt capture, printable documents, the electronic-mail interface, presence in other applications, routes, import and export, and field-level visibility |
| [`acceptance-criteria.md`](acceptance-criteria.md) | The standing fixture and one hundred and thirty-one numbered Given–When–Then scenarios with concrete records, amounts and messages |
| [`glossary.md`](glossary.md) | Every term of the domain, defined, with the stored values behind them and the terms this folder deliberately does not use |

There are no extra topic files: the eleven documents above are the whole folder.

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
